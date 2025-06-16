#  extends

当我们获取到某个Class对象时，实际上就获取到了一个类的类型：
Class cls = String.class; // 获取到String的Class

还可以用实例的getClass()方法获取：
String s = "";
Class cls = s.getClass(); // s是String，因此获取到String的Class

最后一种获取Class的方法是通过Class.forName("")，传入Class的完整类名获取：
Class s = Class.forName("java.lang.String");

这三种方式获取的Class实例都是同一个实例，因为JVM对每个加载的Class只创建一个Class实例来表示它的类型。

## 获取父类的Class
/ reflection
// reflection
public class Main {
    public static void main(String[] args) throws Exception {
        Class i = Integer.class;
        Class n = i.getSuperclass();
        System.out.println(n);
        Class o = n.getSuperclass();
        System.out.println(o);
        System.out.println(o.getSuperclass());
    }
}

运行上述代码，可以看到，
Integer的父类类型是Number，Number的父类是Object，Object的父类是null。
除Object外，其他任何非interface的Class都必定存在一个父类类型。









































































































