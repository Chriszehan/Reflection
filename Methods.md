# Method
我们已经能通过Class实例获取所有Field对象，
同样的，可以通过Class实例获取所有Method信息。

Class类提供了以下几个方法来获取Method：
Method getMethod(name, Class...)：获取某个public的Method（包括父类）
Method getDeclaredMethod(name, Class...)：获取当前类的某个Method（不包括父类）
Method[] getMethods()：获取所有public的Method（包括父类）
Method[] getDeclaredMethods()：获取当前类的所有Method（不包括父类）

我们来看一下示例代码：
// reflection
public class Main {
    public static void main(String[] args) throws Exception {
        Class stdClass = Student.class;
        // 获取public方法getScore，参数为String:
        System.out.println(stdClass.getMethod("getScore", String.class));
        // 获取继承的public方法getName，无参数:
        System.out.println(stdClass.getMethod("getName"));
        // 获取private方法getGrade，参数为int:
        System.out.println(stdClass.getDeclaredMethod("getGrade", int.class));
    }
}

class Student extends Person {
    public int getScore(String type) {
        return 99;
    }
    private int getGrade(int year) {
        return 1;
    }
}

class Person {
    public String getName() {
        return "Person";
    }
}

上述代码首先获取Student的Class实例，
然后，分别获取public方法、继承的public方法以及private方法，
打印出的Method类似：
public int Student.getScore(java.lang.String)
public java.lang.String Person.getName()
private int Student.getGrade(int)

一个Method对象包含一个方法的所有信息：

getName()：返回方法名称，例如："getScore"；
getReturnType()：返回方法返回值类型，也是一个Class实例，例如：String.class；
getParameterTypes()：返回方法的参数类型，是一个Class数组，例如：{String.class, int.class}；
getModifiers()：返回方法的修饰符，它是一个int，不同的bit表示不同的含义。

# 调用方法
当我们获取到一个Method对象时，就可以对它进行调用。我们以下面的代码为例：
String s = "Hello world";
String r = s.substring(6); // "world"

如果用反射来调用substring方法，需要以下代码：
// reflection
import java.lang.reflect.Method;

public class Main {
    public static void main(String[] args) throws Exception {
        // String对象:
        String s = "Hello world";
        // 获取String substring(int)方法，参数为int:
        Method m = String.class.getMethod("substring", int.class);
        // 在s对象上调用该方法并获取结果:
        String r = (String) m.invoke(s, 6);
        // 打印调用结果:
        System.out.println(r); // "world"
    }
}

注意到substring()有两个重载方法，
我们获取的是String substring(int)这个方法。
思考一下如何获取String substring(int, int)方法。

对Method实例调用invoke就相当于调用该方法，
invoke的第一个参数是对象实例，即在哪个实例上调用该方法，
后面的可变参数要与方法参数一致，否则将报错。

# 调用静态方法
如果获取到的Method表示一个静态方法，调用静态方法时，由于无需指定实例对象，
所以invoke方法传入的第一个参数永远为null。
我们以Integer.parseInt(String)为例：

// reflection
import java.lang.reflect.Method;

public class Main {
    public static void main(String[] args) throws Exception {
        // 获取Integer.parseInt(String)方法，参数为String:
        Method m = Integer.class.getMethod("parseInt", String.class);
        // 调用该静态方法并获取结果:
        Integer n = (Integer) m.invoke(null, "12345");
        // 打印调用结果:
        System.out.println(n);
    }
}

# 调用非public方法
和Field类似，对于非public方法，
我们虽然可以通过Class.getDeclaredMethod()获取该方法实例，
但直接对其调用将得到一个IllegalAccessException。
为了调用非public方法，我们通过Method.setAccessible(true)允许其调用：
// reflection
import java.lang.reflect.Method;

public class Main {
    public static void main(String[] args) throws Exception {
        Person p = new Person();
        Method m = p.getClass().getDeclaredMethod("setName", String.class);
        m.setAccessible(true);
        m.invoke(p, "Bob");
        System.out.println(p.name);
    }
}

class Person {
    String name;
    private void setName(String name) {
        this.name = name;
    }
}

此外，setAccessible(true)可能会失败。
如果JVM运行期存在SecurityManager，
那么它会根据规则进行检查，有可能阻止setAccessible(true)。
例如，某个SecurityManager可能不允许对java和javax开头的package的类调用setAccessible(true)，这样可以保证JVM核心库的安全。

# 多态
我们来考察这样一种情况：一个Person类定义了hello()方法，
并且它的子类Student也覆写了hello()方法，
那么，从Person.class获取的Method，作用于Student实例时，
调用的方法到底是哪个？

// reflection
import java.lang.reflect.Method;

public class Main {
    public static void main(String[] args) throws Exception {
        // 获取Person的hello方法:
        Method h = Person.class.getMethod("hello");
        // 对Student实例调用hello方法:
        h.invoke(new Student());
    }
}

class Person {
    public void hello() {
        System.out.println("Person:hello");
    }
}

class Student extends Person {
    public void hello() {
        System.out.println("Student:hello");
    }
}

运行上述代码，发现打印出的是Student:hello，
因此，使用反射调用方法时，仍然遵循多态原则：
即总是调用实际类型的覆写方法（如果存在）。上述的反射代码：
Method m = Person.class.getMethod("hello");
m.invoke(new Student());

实际上相当于：
Person p = new Student();
p.hello();

































































