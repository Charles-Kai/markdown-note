- # springboot源码分析之启动过程

  ## 一、概念

  > 计划写一波springboot 2.x源码分析，只写实用性比较高的特性，从[GitHub](https://github.com/spring-projects/spring-boot/tags?after=v1.0.0.RC3)上看出更新频率在一个月左右，更新极快非常活跃。

  #### 版本发行:

  | 版本      | 时间线     | 说明         |
  | --------- | ---------- | ------------ |
  | v0.5.0.M1 | 2013-08-06 | 第一个版本   |
  | v2.2.0.M3 | 2019-05-15 | 当前最新版本 |

  ## 二、源码分析

  > SpringBoot的启动引导类写法多样，标记了@SpringBootApplication的class作为源类，代码如下：

  ```java
  //简易版
  @SpringBootApplication
   public class MyApplication {
           public static void main(String[] args) {
               SpringApplication.run(MyApplication.class, args);
           }
   }
  
  //通过 SpringApplicationBuilder API
  @SpringBootApplication
  public class DiveInSpringBootApplication {
  
      public static void main(String[] args) {
  
          new SpringApplicationBuilder(DiveInSpringBootApplication.class)
                  .bannerMode(Banner.Mode.CONSOLE)
                  .web(WebApplicationType.NONE)
                  .profiles("prod")
                  .headless(true)
                  .run(args);
      }
  }
  
  //声明new
  public class SpringApplicationBootstrap {
  
      public static void main(String[] args) {
  //        SpringApplication.run(ApplicationConfiguration.class,args);
  
          Set sources = new HashSet();
          // 配置Class 名称
          sources.add(ApplicationConfiguration.class.getName());
          SpringApplication springApplication = new SpringApplication();
          //配置源
          springApplication.setSources(sources);
          //配置控制台banner
          springApplication.setBannerMode(Banner.Mode.CONSOLE);
          //声明web类型
          springApplication.setWebApplicationType(WebApplicationType.NONE);
          //多环境配置激活
          springApplication.setAdditionalProfiles("prod");
          //java.awt.headless禁用模式
          springApplication.setHeadless(true);
          springApplication.run(args);
  
      }
  
      @SpringBootApplication
      public static class ApplicationConfiguration {
  
      }
  
  }
  ```

  > 从代码上可以看出，调用了SpringApplication的静态方法run。这个run方法会构造一个SpringApplication的实例，然后再调用这里实例的run方法就表示启动SpringBoot。因此，想要分析SpringBoot的启动过程，我们需要熟悉SpringApplication的构造过程以及SpringApplication的run方法执行过程即可。

  ### SpringApplication的准备过程

  - 配置 Spring Boot Bean 源：Java 配置 Class 或 XML 上下文配置文件集合，用于 Spring Boot BeanDefinitionLoader 读取 ，并且将配置源解析加载为Spring Bean 定义。

  - 推断 Web 应用类型：根据当前应用 ClassPath 中是否存在相关实现类来推断 Web 应用的类型。参考方法：org.springframework.boot.SpringApplication#deduceWebApplicationType。

    ```java
      private WebApplicationType deduceWebApplicationType() {
          if (ClassUtils.isPresent(REACTIVE_WEB_ENVIRONMENT_CLASS, null)
          	&& !ClassUtils.isPresent(MVC_WEB_ENVIRONMENT_CLASS, null)) {
          	return WebApplicationType.REACTIVE;
          }
          for (String className : WEB_ENVIRONMENT_CLASSES) {
              if (!ClassUtils.isPresent(className, null)) {
              	return WebApplicationType.NONE;
              }
          }
          return WebApplicationType.SERVLET;
      }
    ```

  - 推断引导类（Main Class）：根据 Main 线程执行堆栈判断实际的引导类。参考方法： org.springframework.boot.SpringApplication#deduceMainApplicationClass

    ```java
      private Class deduceMainApplicationClass() {
              try {
      //获取堆栈输出方法名称
                  StackTraceElement[] stackTrace = new RuntimeException().getStackTrace();
                  for (StackTraceElement stackTraceElement : stackTrace) {
                      if ("main".equals(stackTraceElement.getMethodName())) {
                          return Class.forName(stackTraceElement.getClassName());
                      }
                  }
              }
              catch (ClassNotFoundException ex) {
                  // Swallow and continue
              }
              return null;
          }
    ```

  - 加载应用上下文初始器 （ ApplicationContextInitializer ）：利用 Spring 工厂加载机制，实例化 ApplicationContextInitializer 实现类，并排序对象集合。

    ```java
      //实现类： org.springframework.core.io.support.SpringFactoriesLoader
      //配置资源： META-INF/spring.factories
      //排序： AnnotationAwareOrderComparator#sort
      private  Collection getSpringFactoriesInstances(Class type,
                  Class[] parameterTypes, Object... args) {
              ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
              // Use names and ensure unique to protect against duplicates
              Set names = new LinkedHashSet<>(
                      SpringFactoriesLoader.loadFactoryNames(type, classLoader));
              List instances = createSpringFactoriesInstances(type, parameterTypes,
                      classLoader, args, names);
              AnnotationAwareOrderComparator.sort(instances);
              return instances;
          }·
    ```

  - 加载应用事件监听器（ ApplicationListener ）：利用 Spring 工厂加载机制，实例化 ApplicationListener 实现类，并排序对象集合。

  ### SpringApplication 运行阶段

  - 加载 SpringApplication 运行监听器（ SpringApplicationRunListeners ）：利用 Spring 工厂加载机制，读取 SpringApplicationRunListener 对象集合，并且封装到组合类SpringApplicationRunListeners。
  - 运行 SpringApplication 运行监听器（ SpringApplicationRunListeners ）：
    1. started(run方法执行的时候立马执行；对应事件的类型是ApplicationStartedEvent)
    2. environmentPrepared(ApplicationContext创建之前并且环境信息准备好的时候调用；对应事件的类型是ApplicationEnvironmentPreparedEvent)
    3. contextPrepared(ApplicationContext创建好并且在source加载之前调用一次；没有具体的对应事件)
    4. contextLoaded(ApplicationContext创建并加载之后并在refresh之前调用；对应事件的类型是ApplicationPreparedEvent)
    5. finished(run方法结束之前调用；对应事件的类型是ApplicationReadyEvent或ApplicationFailedEvent)
  - 创建 Spring 应用上下文（ ConfigurableApplicationContext ）:根据准备阶段的推断 Web 应用类型创建对应ConfigurableApplicationContext 实例：
    - Web Reactive： AnnotationConfigReactiveWebServerApplicationContext
    - Web Servlet： AnnotationConfigServletWebServerApplicationContext
    - 非 Web： AnnotationConfigApplicationContext
  - 创建 Environment：根据准备阶段的推断 Web 应用类型创建对应的 ConfigurableEnvironment 实例。
    - Web Reactive： StandardEnvironment
    - Web Servlet： StandardServletEnvironment
    - 非 Web： StandardEnvironment

  ### run方法分析

  ```
  public ConfigurableApplicationContext run(String... args) {
    StopWatch stopWatch = new StopWatch(); // 构造一个任务执行观察器
    stopWatch.start(); // 开始执行，记录开始时间
    ConfigurableApplicationContext context = null;
    configureHeadlessProperty();
    // 获取SpringApplicationRunListeners，内部只有一个EventPublishingRunListener
    SpringApplicationRunListeners listeners = getRunListeners(args);
    // 上面分析过，会封装成SpringApplicationEvent事件然后广播出去给SpringApplication中的listeners所监听
    // 这里接受ApplicationStartedEvent事件的listener会执行相应的操作
    listeners.started();
    try {
      // 构造一个应用程序参数持有类
      ApplicationArguments applicationArguments = new DefaultApplicationArguments(
          args);
      // 创建Spring容器
      context = createAndRefreshContext(listeners, applicationArguments);
      // 容器创建完成之后执行额外一些操作
      afterRefresh(context, applicationArguments);
      // 广播出ApplicationReadyEvent事件给相应的监听器执行
      listeners.finished(context, null);
      stopWatch.stop(); // 执行结束，记录执行时间
      if (this.logStartupInfo) {
        new StartupInfoLogger(this.mainApplicationClass)
            .logStarted(getApplicationLog(), stopWatch);
      }
      return context; // 返回Spring容器
    }
    catch (Throwable ex) {
      handleRunFailure(context, listeners, ex); // 这个过程报错的话会执行一些异常操作、然后广播出ApplicationFailedEvent事件给相应的监听器执行
      throw new IllegalStateException(ex);
    }
  }
  ```

  ## 反馈与建议

  - 为了快大多分析的不好写的很乱，凑合看下我以后改下排版😂。
  - 邮箱：[caochikai@qq.com](

- mailto:caochikai@qq.com)
