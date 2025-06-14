# 字段
对任意的一个Object实例，只要我们获取了它的Class，就可以获取它的一切信息。

我们先看看如何通过Class实例获取字段信息。Class类提供了以下几个方法来获取字段：

Field getField(name)：根据字段名获取某个public的field（包括父类）
Field getDeclaredField(name)：根据字段名获取当前类的某个field（不包括父类）
Field[] getFields()：获取所有public的field（包括父类）
Field[] getDeclaredFields()：获取当前类的所有field（不包括父类）

我们来看一下示例代码：

// reflection
public class Main {
    public static void main(String[] args) throws Exception {
        Class stdClass = Student.class;
        // 获取public字段"score":
        System.out.println(stdClass.getField("score"));
        // 获取继承的public字段"name":
        System.out.println(stdClass.getField("name"));
        // 获取private字段"grade":
        System.out.println(stdClass.getDeclaredField("grade"));
    }
}

class Student extends Person {
    public int score;
    private int grade;
}

class Person {
    public String name;
}

上述代码首先获取Student的Class实例，
然后，分别获取public字段、继承的public字段以及private字段，
打印出的Field类似：

public int Student.score
public java.lang.String Person.name
private int Student.grade

一个Field对象包含了一个字段的所有信息：
getName()：返回字段名称，例如，"name"；
getType()：返回字段类型，也是一个Class实例，例如，String.class；
getModifiers()：返回字段的修饰符，它是一个int，不同的bit表示不同的含义。

以String类的value字段为例，它的定义是：
public final class String {
    private final byte[] value;
}

我们用反射获取该字段的信息，代码如下：
Field f = String.class.getDeclaredField("value");
f.getName(); // "value"
f.getType(); // class [B 表示byte[]类型
int m = f.getModifiers();
Modifier.isFinal(m); // true
Modifier.isPublic(m); // false
Modifier.isProtected(m); // false
Modifier.isPrivate(m); // true
Modifier.isStatic(m); // false

# 获取字段值
利用反射拿到字段的一个Field实例只是第一步，我们还可以拿到一个实例对应的该字段的值。

例如，对于一个Person实例，我们可以先拿到name字段对应的Field，
再获取这个实例的name字段的值：

// reflection
import java.lang.reflect.Field;
public class Main {

    public static void main(String[] args) throws Exception {
        Object p = new Person("Xiao Ming");
        Class c = p.getClass();
        Field f = c.getDeclaredField("name");
        Object value = f.get(p);
        System.out.println(value); // "Xiao Ming"
    }
}

class Person {
    private String name;

    public Person(String name) {
        this.name = name;
    }
}

上述代码先获取Class实例，再获取Field实例，
然后，用Field.get(Object)获取指定实例的指定字段的值。

运行代码，如果不出意外，会得到一个IllegalAccessException，
这是因为name被定义为一个private字段，
正常情况下，Main类无法访问Person类的private字段。
要修复错误，可以将private改为public，
或者，在调用Object value = f.get(p);前，
先写一句：

f.setAccessible(true);
调用Field.setAccessible(true)的意思是，
别管这个字段是不是public，一律允许访问。

可以试着加上上述语句，再运行代码，就可以打印出private字段的值。

有童鞋会问：如果使用反射可以获取private字段的值，那么类的封装还有什么意义？

答案是正常情况下，我们总是通过p.name来访问Person的name字段，
编译器会根据public、protected和private决定是否允许访问字段，
这样就达到了数据封装的目的。

而反射是一种非常规的用法，使用反射，
首先代码非常繁琐，
其次，它更多地是给工具或者底层框架来使用，
目的是在不知道目标实例任何信息的情况下，获取特定字段的值。

此外，setAccessible(true)可能会失败。
如果JVM运行期存在SecurityManager，
那么它会根据规则进行检查，有可能阻止setAccessible(true)。
例如，某个SecurityManager可能不允许对java和javax开头的package的类调用setAccessible(true)，
这样可以保证JVM核心库的安全。

# 设置字段值

通过Field实例既然可以获取到指定实例的字段值，自然也可以设置字段的值。

设置字段值是通过Field.set(Object, Object)实现的，
其中第一个Object参数是指定的实例，第二个Object参数是待修改的值。示例代码如下：

// reflection
import java.lang.reflect.Field;

public class Main {

    public static void main(String[] args) throws Exception {
        Person p = new Person("Xiao Ming");
        System.out.println(p.getName()); // "Xiao Ming"
        Class c = p.getClass();
        Field f = c.getDeclaredField("name");
        f.setAccessible(true);
        f.set(p, "Xiao Hong");
        System.out.println(p.getName()); // "Xiao Hong"
    }
}

class Person {
    private String name;

    public Person(String name) {
        this.name = name;
    }

    public String getName() {
        return this.name;
    }
}

运行上述代码，打印的name字段从Xiao Ming变成了Xiao Hong，
说明通过反射可以直接修改字段的值。

同样的，修改非public字段，需要首先调用setAccessible(true)。










































