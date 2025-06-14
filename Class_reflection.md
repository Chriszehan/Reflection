# Class

除了int等基本类型外，Java的其他类型全部都是class（包括interface）。例如：

String
Object
Runnable
Exception

仔细思考，我们可以得出结论：class（包括interface）的本质是数据类型（Type）。
无继承关系的数据类型无法赋值：

Number n = new Double(123.456); // OK
String s = new Double(123.456); // compile error!

而class是由JVM在执行过程中动态加载的。
JVM在第一次读取到一种class类型时，将其加载进内存。




























































































