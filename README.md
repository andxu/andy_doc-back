## JDWP and JDI



JVM TI - Java VM Tool Interface
    Defines the debugging services a JVM provides.
JDWP - Java Debug Wire Protocol
    Defines the communication between debuggee and debugger processes.
JDI - Java Debug Interface
    Defines a high-level Java language interface which tool developers can easily use to write remote debugger applications.

the modern JDWP agent was embeded in JVM since 1999, java 1.2.2, nearly twenty years ago,  with minor changes to support languange features in each java versions, it is developed using c++ languange and it supports transporting data using sockets and shared memories. It has nearly 90 commands each of them is designed to fullfill a very simple task, for example: get the thread name for a specifed thread id. JDI is a serries of interfaces and JDK has also a standard implementation of it, eclipse has also another implementation of it, using JDI, you are free of constructing JDWP requests and decoding JDWP reponses. You might not know that the JDWP agent is single threaded with a request buffer and a repsonse buffer, both the JDI implementations are using sync mode because async is not invented by the the time, that means each JDWP command will need to wait for the response of previous request even when there is no depenceny between them. Sync mode and fragmented request/response make the major latency issue of java remote debugging over the internet. 



```

                /    |--------------|
               /     |     JVM      |
 target   ----(      |--------------|  <------- JVM TI - Java JVM Tool Interface
               \     |   back-end   |
                \    |--------------|
                /           |
 comm channel -(            |  <--------------- JDWP - Java Debug Wire Protocol
                \           |
                     |--------------|
                     | front-end    |
                     |--------------|  <------- JDI - Java Debug Interface
                     |      UI      |
                     |--------------|
```


    J2SE 1.2.2 Cricket 蟋蟀 1999-07-08
    J2SE 1.3 Kestrel 美洲红隼 2000-05-08
    J2SE 1.3.1 Ladybird 瓢虫 2001-05-17
    J2SE 1.4.0 Merlin 灰背隼 2002-02-13
    J2SE 1.4.1 grasshopper 蚱蜢 2002-09-16
    J2SE 1.4.2 Mantis 螳螂 2003-06-26
    J2SE 5.0 (1.5.0) Tiger 老虎 2004-10
    J2SE 6.0 (Beta) Mustang 野马 2006-04




On the contrary, the node.js and c# debug protocol has polished the debug protocol for remote debugging, and we have proven vscode protocol is more efficient to fullfill debug needs. Let me give a comparasion of these two protocols using a typical spring boot application on four typical scenarios.


Case 1: set on breakpoint


name | count | request size | response size| total size
---- | --- | --- | --- | ---
linetable|2|76|146|222
set|3|164|45|209
methodswithgeneric|1|30|97|127
classesbysignature|1|49|28|77
capabilitiesnew|1|22|43|65
sourcedebugextension|1|30|11|41
*total*|9|371|370|741
*vscode*|1(11.11 %)|252(67.92 %)|158(42.70 %)|410(55.33 %)

Case 2: threads

name | count | request size | response size| total size
---- | --- | --- | --- | ---
thread::name|22|660|804|1464
iscollected|22|660|264|924
allthreads|1|22|191|213
set|2|56|30|86
*total*|47|1398|1289|2687
*vscode*|1(2.13 %)|46(3.29 %)|1203(93.33 %)|1249(46.48 %)

Case 3: get stack trace

name | count | request size | response size| total size
---- | --- | --- | --- | ---
methodswithgeneric|36|1080|59189|60269
linetable|18|684|5130|5814
frames|1|38|2226|2264
signature|12|360|796|1156
sourcefile|12|360|486|846
sourcedebugextension|11|330|121|451
allthreads|1|22|191|213
framecount|1|30|15|45
iscollected|1|30|12|42
*total*|93|2934|68166|71100
*vscode*|1(1.08 %)|104(3.54 %)|8769(12.86 %)|8873(12.48 %)

Case 4: evaluation on 'this.hashCode()'

name | count | request size | response size| total size
---- | --- | --- | --- | ---
frames|4|152|6726|6878
fieldswithgeneric|5|150|1839|1989
sourcedebugextension|25|750|275|1025
methodswithgeneric|1|30|585|615
superclass|6|180|114|294
interfaces|5|150|83|233
variabletablewithgeneric|1|38|134|172
referencetype|2|60|40|100
parent|2|60|38|98
thread::status|2|60|38|98
object::invokemethod|1|62|25|87
object::getvalues|1|42|17|59
thisobject|1|38|20|58
threadgroup|1|30|19|49
threadgroup::name|1|30|19|49
classloader|1|30|19|49
*total*|59|1862|9991|11853
*vscode*|1(1.69 %)|123(6.61 %)|169(1.69 %)|292(2.46 %)


As a conclusion, the overall JDWP request/response numbers is over 50 times as vscode, the overall JDWP request/response body is 3 times as vsocde, producing overy 40 times of lentency of each operation. The solution is to replace the JDWP protocol over the internet with vscode protocol. In order to archive this, we make a minior change on the archecture diagram, we move the debug core module which is to comunicate with JVM  from local to the remote container or vm where the JVM runs at, we call it debug agent server,  we also use a agent client at local to communicate with debug agent server using vscode debug protocol.

Now let me demo the debug experience using debug agent.
I have setup a spring boot application, set a breakpoint, now let me start the debug session, we can see that the breakpoint is hit, step over and over, we can see that the operation can be completed in less than 1 seconds.

# Problems
1. We need to brand new a debug server supporting this archetecture. Because one vscode debug operation like set breakpoint also needs to interact with JDT language server and local project file by debug provider described before, it is hard to have one debug server which supports both local debug and agent debug.
2. We cannot easily implement evaluation and HCR, because the evaluation/hcr provider needs the project support to generate java bytecode instructions, so it cannot be moved to remote together with debug core, it requires us to extends vscode protocol for transport java bytecode instructions. 
All reasons above will result in a huge change to our code, then why we cannot move the entire debug server to remote? That is because the debug server depends on JDT language server for project support, if we move all debug server to remote, we also need to setup the project remote the same as the local ones, which means your project in remote must have the exact same path as your local machine, and you cannot move/rename your local project, the jdk version and location must be same too and the third-party librairies.

