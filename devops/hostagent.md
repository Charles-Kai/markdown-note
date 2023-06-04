https://github.com/everydoc/jrebel-license-server
http://idea.javatiku.cn/
https://www.cnblogs.com/joyny/p/10237136.html
http://jrebel.yanjiayu.cn:9001/

aven集成HotswapAgent，热加载指定的目录

2023/6/4 17:45:19

要使用 Maven 集成 HotswapAgent 实现热加载指定目录下的类，需要进行以下步骤：

在 pom.xml 文件中添加 HotswapAgent 的依赖：
xml
<dependency>
    <groupId>org.hotswapagent</groupId>
    <artifactId>hotswap-agent-core</artifactId>
    <version>1.4.0</version>
    <scope>runtime</scope>
</dependency>
在启动命令中添加 HotswapAgent 的 JVM 参数，例如：
java -javaagent:/path/to/hotswap-agent.jar -Dhotswap.scan.dir=/path/to/classes -jar myapp.jar
其中 -javaagent:/path/to/hotswap-agent.jar 是启用 HotswapAgent 的 JVM 参数，-Dhotswap.scan.dir=/path/to/classes 是指定需要热加载的目录，即 /path/to/classes 目录下的类文件将会被监控并进行热加载。

在启用了 HotswapAgent 的应用程序中使用热加载功能。例如，可以创建一个简单的类，并在其中调用需要热加载的类的方法：
java
public class MyApp {
    public static void main(String[] args) throws Exception {
        MyClass myClass = new MyClass();
        while (true) {
            myClass.sayHello(); // 调用需要热加载的类的方法
            Thread.sleep(5000L);
        }
    }
}
修改需要热加载的类，并观察修改是否生效。例如，可以修改 MyClass 类的实现：
java
public class MyClass {
    public void sayHello() {
        System.out.println("Hello, world!");
    }
}
将其修改为：

java
public class MyClass {
    public void sayHello() {
        System.out.println("Hello, HotswapAgent!");
    }
}
保存并编译该类，HotswapAgent 将会自动检测到该类的变化并进行热加载。重新运行程序后，就可以看到输出已经变成了“Hello, HotswapAgent!”。

注意，使用 HotswapAgent 进行热加载需要遵循一定的规则和限制，例如只能替换方法体相同的方法，不能增加或删除方法等。在实际应用中需要根据具体情况选择适合的方案。

不通过启动命令，集成在当前maven项目代码里面

2023/6/4 17:47:38

如果你想在当前 Maven 项目代码里面集成 HotswapAgent，可以按照以下步骤进行操作：

在 pom.xml 文件中添加 HotswapAgent 的依赖：
xml
<dependency>
    <groupId>org.hotswapagent</groupId>
    <artifactId>hotswap-agent-core</artifactId>
    <version>1.4.0</version>
    <scope>runtime</scope>
</dependency>
创建一个 HotswapClassLoader 类，并继承自 URLClassLoader。该类用于加载热加载的类文件。
java
public class HotswapClassLoader extends URLClassLoader {
    public HotswapClassLoader(ClassLoader parent) {
        super(new URL[0], parent);
    }

    public void addURL(URL url) {
        super.addURL(url);
    }
}
在需要热加载的类中使用该 HotswapClassLoader 类加载热加载的类文件，并调用 HotswapAgent 的 reload() 方法进行热加载。例如：
java
public class MyApp {
    public static void main(String[] args) throws Exception {
        while (true) {
            // 创建一个新的 HotswapClassLoader 实例
            HotswapClassLoader classLoader = new HotswapClassLoader(MyApp.class.getClassLoader());

            // 加载需要热加载的类
            Class<?> clazz = classLoader.loadClass("MyClass");

            // 获取当前线程的 HotswapAgent 实例
            HotswapAgent agent = HotswapAgent.getInstrumentation();

            // 调用 HotswapAgent 的 reload() 方法进行热加载
            agent.reload(new ClassDefinition(clazz, classBytes()));

            // 使用反射机制实例化和调用新版本的类
            Object instance = clazz.newInstance();
            Method method = clazz.getMethod("sayHello");
            method.invoke(instance);

            // 等待一段时间后继续热加载
            Thread.sleep(5000L);
        }
    }

    private static byte[] classBytes() throws Exception {
        // 读取需要热加载的类字节码文件并返回
        InputStream is = MyApp.class.getResourceAsStream("/MyClass.class");
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        byte[] buffer = new byte[4096];
        int len;
        while ((len = is.read(buffer)) > 0) {
            bos.write(buffer, 0, len);
        }
        return bos.toByteArray();
    }
}
在上面的例子中，我们首先创建了一个新的 HotswapClassLoader 实例，并使用它加载了需要热加载的类。然后获取当前线程的 HotswapAgent 实例，并调用其 reload() 方法进行热加载。

注意，在这种情况下，热加载的类文件需要放在 Maven 项目的 resources 目录下，否则无法正确加载。另外，由于需要使用反射机制实例化和调用新版本的类，因此需要确保新版本的类具有与旧版本相同的构造函数和方法。




 
