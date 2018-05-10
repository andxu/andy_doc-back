
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
