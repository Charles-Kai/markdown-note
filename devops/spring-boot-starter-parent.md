如果您想在子工程中通过dependency标签覆盖父工程的spring-boot-starter-parent版本，可以在子工程的pom.xml文件中声明一个新的<parent>标签，并将<relativePath>元素指向父工程的pom.xml文件。然后，在子工程中添加一个dependencyManagement标签，并将要覆盖的依赖库的版本号指定为一个属性。

具体的步骤如下：

在子工程的<project>标签中加入以下内容：
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.0</version> <!-- 新的版本号 -->
    <relativePath>../pom.xml</relativePath> <!-- 指向父工程的pom.xml文件 -->
</parent>
在子工程中添加一个dependencyManagement标签，指定要覆盖的依赖库的版本号为一个属性，如下所示：
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>${spring-boot.version}</version> <!-- 将版本号指定为一个属性 -->
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<properties>
    <spring-boot.version>2.5.0</spring-boot.version> <!-- 定义属性，表示新的版本号 -->
</properties>
这样配置之后，子工程中所有的Spring Boot依赖库都会使用指定的版本号，而不是从父工程继承的默认版本号。