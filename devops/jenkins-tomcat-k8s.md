maven build war and jar in package.

# k8s-jenkins持续发布tomcat项目

[ k8s-jenkins持续发布tomcat项目]([k8s-jenkins持续发布tomcat项目 - windysai - 博客园 (cnblogs.com)](https://www.cnblogs.com/windysai/p/14327932.html))

[利用configmap覆盖tomcat项目数据库配置文件 - windysai - 博客园 (cnblogs.com)](https://www.cnblogs.com/windysai/p/14305441.html)

[springboot同时打jar包和war包_pom兼容springboot能打jar也可以打war-CSDN博客](https://blog.csdn.net/ThinkPet/article/details/106180111)

[03-3-Pod控制器 (yuque.com)](https://www.yuque.com/duduniao/k8s/hlzt2y)

[NB Tech Support / Devops · GitLab](https://gitlab.com/nb-tech-support/devops)

[GitHub - sheehan/job-dsl-gradle-example: An example Job DSL project that uses Gradle for building and testing.](https://github.com/sheehan/job-dsl-gradle-example)

[GitHub - jenkinsci/pipeline-examples: A collection of examples, tips and tricks and snippets of scripting for the Jenkins Pipeline plugin](https://github.com/jenkinsci/pipeline-examples)

[GitHub - wcm-io-devops/jenkins-pipeline-library: wcm.io Jenkins Pipeline Library for CI/CD](https://github.com/wcm-io-devops/jenkins-pipeline-library)

Spring Boot可以同时生成JAR包和WAR包，但通常情况下，选择生成JAR包或WAR包取决于你的应用程序部署的环境。如果你要部署到独立的容器（如Tomcat之类的Servlet容器），则需要生成WAR包。如果你想要使用Spring Boot的内嵌式容器（如内嵌的Tomcat或Jetty），则生成JAR包更为方便。

要同时生成JAR包和WAR包，你可以在Spring Boot项目的pom.xml文件中进行配置。下面是一个示例：

```xml
<build>
    <plugins>
        <!-- Maven插件配置，用于构建JAR包 -->
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>${spring-boot.version}</version>
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
        <!-- Maven插件配置，用于构建WAR包 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-war-plugin</artifactId>
            <version>3.2.2</version>
            <configuration>
                <failOnMissingWebXml>false</failOnMissingWebXml>
            </configuration>
        </plugin>
    </plugins>
</build>
```

在这个配置中，`spring-boot-maven-plugin`用于构建JAR包，而`maven-war-plugin`用于构建WAR包。确保配置中的版本号与你当前使用的Spring Boot和Maven版本兼容。