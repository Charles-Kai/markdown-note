https://github.com/everydoc/jrebel-license-server
http://idea.javatiku.cn/
https://www.cnblogs.com/joyny/p/10237136.html
http://jrebel.yanjiayu.cn:9001/

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




 
