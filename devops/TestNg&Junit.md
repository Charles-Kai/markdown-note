如何使用maven-surefire-plugin在同一个项目中执行JUnit和TestNG测试？
https://solidsoft.wordpress.com/2013/03/12/mixing-testng-and-junit-tests-in-one-maven-module-2013-edition/

https://maven.apache.org/surefire/maven-surefire-plugin/examples/testng.html

您可以使用Maven来同时集成TestNG和JUnit的项目。以下是一些步骤：

在pom.xml文件中添加以下依赖项：
XML
AI 生成的代码。仔细查看和使用。 有关常见问题解答的详细信息.

<dependency>
    <groupId>org.testng</groupId>
    <artifactId>testng</artifactId>
    <version>7.5.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
</dependency>
在pom.xml文件中添加以下插件：
XML
AI 生成的代码。仔细查看和使用。 有关常见问题解答的详细信息.

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.0.0-M5</version>
    <dependencies>
        <dependency>
            <groupId>org.testng</groupId>
            <artifactId>testng</artifactId>
            <version>7.5.0</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
        </dependency>
    </dependencies>
</plugin>
在src/test/java目录下创建JUnit测试类和TestNG测试类。

运行以下命令来执行测试：

mvn test

这将同时运行JUnit和TestNG测试。