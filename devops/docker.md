https://blog.csdn.net/alinyua/article/details/81094240

2023/6/9 23:25:01

springboot项目通过maven插件复制打包jar，配置环境变量，并使用宿主机的端口，贡献网络，打包成docker镜像并执行docker run

2023/6/9 23:25:27

可以通过以下步骤在Spring Boot项目中使用Maven插件将其打包成Docker镜像并运行：

在pom.xml文件中添加以下插件：
<plugin>
  <groupId>com.spotify</groupId>
  <artifactId>docker-maven-plugin</artifactId>
  <version>1.0.0</version>
  <configuration>
    <imageName>your-docker-image-name</imageName>
    <dockerDirectory>${project.basedir}/src/main/docker</dockerDirectory>
    <resources>
      <resource>
        <targetPath>/</targetPath>
        <directory>${project.build.directory}</directory>
        <include>${project.build.finalName}.jar</include>
      </resource>
    </resources>
  </configuration>
</plugin>
其中，imageName指定了生成的Docker镜像名称，${project.build.finalName}.jar表示要打包的jar包名称。

在项目根目录下创建src/main/docker目录，并在该目录下创建一个名为Dockerfile的文件，内容如下：
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ARG JAR_FILE
COPY ${JAR_FILE} app.jar
ENV PORT=8080
EXPOSE $PORT
ENTRYPOINT ["java","-jar","/app.jar"]
Dockerfile中定义了Docker镜像的构建规则，从官方的Java 8镜像中开始构建，复制打包好的应用jar包到容器内，设置环境变量端口号为8080，暴露端口并执行启动命令。

执行mvn clean package docker:build命令，Maven将会编译打包应用程序并构建Docker镜像。

执行docker run -p 8080:8080 your-docker-image-name命令启动Docker容器，并将容器的8080端口映射到宿主机的8080端口，从而使得我们可以通过宿主机的IP地址和8080端口访问应用程序。

访问http://localhost:8080 即可在本地浏览器上查看该项目的运行结果。




2023/6/9 23:25:52

这个插件执行会卡住

2023/6/9 23:26:10

如果使用 Docker Maven 插件时出现卡住的情况，可能是因为该插件需要下载 Docker 镜像来进行打包和构建。您可以尝试以下方法解决该问题：

确认您的网络连接正常。在执行 Maven 命令之前，可以尝试通过 docker pull 命令手动下载相关的 Docker 镜像来测试网络连接。

检查 Maven 的日志输出，查看是否存在关于 Docker 容器的错误或警告信息。可以使用 -X 参数来开启 Maven 的详细输出。

尝试增加 Maven 插件的超时时间。在 Maven 的 settings.xml 文件中添加以下配置：

<build>
  <plugins>
    <plugin>
      <groupId>com.spotify</groupId>
      <artifactId>docker-maven-plugin</artifactId>
      <configuration>
        <connectionTimeout>300000</connectionTimeout>
        <readTimeout>900000</readTimeout>
      </configuration>
    </plugin>
  </plugins>
</build>
其中 connectionTimeout 和 readTimeout 分别指定了插件的连接和读取超时时间，单位为毫秒。您可以根据实际情况调整这些值。

如果上述方法都无法解决问题，可以尝试重新安装 Docker 或更新 Docker 版本。也可以尝试使用其他类似的 Docker 插件来进行打包和构建。




2023/6/9 23:26:22

还有其他docker插件吗？

2023/6/9 23:26:42

是的，除了 Docker Maven 插件以外，还有其他一些常用的 Docker 插件可以用来将应用程序打包成 Docker 镜像。以下是其中几个比较受欢迎的插件：

Google Jib：官方文档称其为“无需 Docker 守护进程的 Java 应用容器构建工具”，使用 Jib 可以轻松地将 Java 应用打包成独立的 Docker 镜像，不需要安装 Docker 工具或 daemon。Jib 还支持将镜像直接推送到 Docker Hub 或其他容器镜像仓库。

Fabric8 Maven 插件：该插件提供了一组命令行工具和 Maven 目标，可以将 Java 应用程序转换为 Docker 镜像，并将其部署到 Kubernetes 等容器编排平台上。Fabric8 插件还提供了一些其他实用功能，例如自动化部署、服务发现和负载均衡等。

Spotify Dockerfile Maven 插件：与 Docker Maven 插件类似，该插件允许在 Maven 构建过程中自定义 Dockerfile 文件，并将应用程序打包成 Docker 镜像。Spotify 插件还支持 Dockerfile 模板、Docker Compose 和多阶段构建等高级特性。

这些插件都具有各自的特点和优势，您可以根据项目需求选择适合的插件。




2023/6/9 23:27:15

请使用Fabric8 Maven 插件实现上面的要求

2023/6/9 23:27:57

好的，下面是使用 Fabric8 Maven 插件将 Spring Boot 项目打包成 Docker 镜像并运行的步骤：

在 pom.xml 文件中添加 Fabric8 Maven 插件依赖：
<dependency>
    <groupId>io.fabric8</groupId>
    <artifactId>fabric8-maven-plugin</artifactId>
    <version>5.6.0</version>
    <scope>provided</scope>
</dependency>
在 pom.xml 文件的 build 节点中添加以下配置：
<build>
    <plugins>
        <!-- ... 其他插件 ... -->
        <plugin>
            <groupId>io.fabric8</groupId>
            <artifactId>fabric8-maven-plugin</artifactId>
            <version>5.6.0</version>
            <executions>
                <execution>
                    <id>fmp</id>
                    <goals>
                        <goal>resource</goal>
                        <goal>build</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <docker>
                    <name>${project.artifactId}</name>
                    <publishRegistry>
                        <host>docker.io</host>
                        <namespace>your-username</namespace>
                    </publishRegistry>
                </docker>
                <images>
                    <image>
                        <name>${project.artifactId}:${project.version}</name>
                        <build>
                            <from>openjdk:8-jdk-alpine</from>
                            <assembly>
                                <descriptorRef>artifact-with-dependencies</descriptorRef>
                            </assembly>
                            <ports>
                                <port>8080</port>
                            </ports>
                        </build>
                    </image>
                </images>
            </configuration>
        </plugin>
    </plugins>
</build>
该配置指定了 Fabric8 Maven 插件的一些参数，其中：

docker 节点用于指定生成的 Docker 镜像的元信息。
publishRegistry 节点用于指定 Docker 镜像发布到的镜像仓库地址和用户名。
images 节点用于指定要构建的 Docker 镜像信息。
build 节点用于指定 Docker 镜像的构建规则，包括基础镜像、文件复制、端口号等。
在此示例中，我们使用 OpenJDK 8 的官方 Alpine 版本作为基础镜像，并将 Spring Boot 项目及其依赖打包成一个可执行 jar 文件。同时，我们还指定了容器暴露的端口号为 8080。

在命令行中执行以下命令：
mvn clean package fabric8:build
该命令会执行 Maven 构建并使用 Fabric8 Maven 插件来构建 Docker 镜像。

执行以下命令运行 Docker 容器：
docker run -p 8080:8080 -d your-username/my-app:1.0.0
其中 -p 参数指定容器端口 8080 映射到宿主机端口 8080，-d 参数指定容器在后台运行，your-username/my-app:1.0.0 指定 Docker 镜像的完整名称。

访问 http://localhost:8080 即可在本地浏览器上查看该项目的运行结果。



2023/6/9 23:28:44

发布镜像地址是本机的docker，没有账户密码限制

2023/6/9 23:28:58

如果您要将 Docker 镜像发布到本地 Docker daemon，可以在 Fabric8 Maven 插件配置中指定 dockerHost 参数，如下所示：

<configuration>
  <dockerHost>unix:///var/run/docker.sock</dockerHost>
  <!-- 其他配置 -->
</configuration>
该配置告诉 Fabric8 Maven 插件使用 Unix 套接字连接到本地 Docker daemon。如果您的 Docker daemon 使用 TCP 连接，请将 dockerHost 参数改为类似 tcp://localhost:2375 的格式。

在执行 mvn clean package fabric8:build 命令时，Fabric8 Maven 插件会自动将构建好的 Docker 镜像上传到 Docker daemon，并可以直接从本地运行该镜像。

例如，以下命令会在本机启动一个新容器并运行 Docker 镜像：

docker run -p 8080:8080 -d my-app:1.0.0
请确保您已经安装了 Docker 环境，并且已经以当前用户身份登录到 Docker daemon 中。如果需要使用账户和密码进行 Docker 登录，请先使用 docker login 命令进行认证。




2023/6/9 23:29:45

docker-maven-plugin也需要制定本地的docker？

2023/6/9 23:30:03

是的，使用 Docker Maven 插件构建 Docker 镜像时，也需要指定本地 Docker daemon 的信息。您可以在插件的配置中添加以下参数来指定 Docker daemon 服务器的地址：

<configuration>
  <dockerHost>unix:///var/run/docker.sock</dockerHost>
  <!-- 其他配置 -->
</configuration>
该配置中的 dockerHost 参数指定了本地 Docker daemon 的 Unix 套接字地址，这意味着 Maven 将通过 Unix 套接字连接到 Docker daemon。如果您的 Docker daemon 使用 TCP 连接，请将该参数改为类似 tcp://localhost:2375 的格式。

在执行 mvn clean package docker:build 命令时，Docker Maven 插件将使用指定的 Docker daemon 构建 Docker 镜像，并将镜像推送到 Docker registry 或者本地 Docker daemon 中。

例如，以下命令会在本机启动一个新容器并运行 Docker 镜像：

docker run -p 8080:8080 -d your-docker-image-name
请确保您已经安装了 Docker 环境，并且已经以当前用户身份登录到 Docker daemon 中。如果需要使用账户和密码进行 Docker 登录，请先使用 docker login 命令进行认证。




2023/6/9 23:31:37

如果maven项目target目录下存在jar，则执行某个shell文件

2023/6/9 23:32:13

您可以使用 Maven 的 Exec 插件来在构建完成后执行 Shell 脚本。具体步骤如下：

在 pom.xml 文件中添加 Exec 插件依赖：
<dependency>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>exec-maven-plugin</artifactId>
    <version>3.0.0</version>
</dependency>
在 pom.xml 文件的 build 节点中添加以下配置：
<build>
    <plugins>
        <!-- ... 其他插件 ... -->
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>exec-maven-plugin</artifactId>
            <version>3.0.0</version>
            <executions>
                <execution>
                    <id>run-script</id>
                    <phase>install</phase>
                    <goals>
                        <goal>exec</goal>
                    </goals>
                    <configuration>
                        <executable>/bin/bash</executable>
                        <arguments>
                            <argument>path/to/your/script.sh</argument>
                        </arguments>
                        <skip>${skip.exec}</skip>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
该配置指定了 Exec 插件的一些参数，其中：

executable 参数指定了要执行的 Shell 程序路径，这里为 /bin/bash，表示使用 Bash 解释器。
arguments 参数指定了要传递给 Shell 程序的参数，这里为脚本文件的路径，您需要将 path/to/your/script.sh 改为自己的脚本文件路径。
skip 参数指定了是否跳过脚本执行，这里使用 Maven 属性 ${skip.exec} 控制，如果该属性被设置为 true，则会跳过脚本执行。
注意：在上述配置中，我们将 Exec 插件的执行阶段设置为 install，表示在项目构建完成后执行 Shell 脚本。如果您需要在其他阶段执行脚本，请将 <phase> 标签改为相应的阶段名称。

在命令行中执行以下命令：
mvn clean install
该命令会执行 Maven 构建和安装，并在安装完成后自动执行 Shell 脚本。

如果您希望只有在 target 目录下存在 jar 文件时才执行 Shell 脚本，可以在脚本中添加逻辑判断，例如：

#!/bin/bash

if [ -f ./target/*.jar ]; then
    # 执行 Jar 包
else
    echo "Jar file not found!"
fi
在该脚本中，我们使用 Shell 的 -f 参数来判断目录下是否存在 jar 文件，如果存在则执行 Jar 包，否则输出错误消息。




2023/6/9 23:32:41

环境是window，安装了git bash，不是linux

2023/6/9 23:32:52

如果您在 Windows 环境下使用 Git Bash，可以将上述 Shell 脚本的可执行权限改为 755，并将 #!/bin/bash 修改为 #!/usr/bin/env bash。

例如，以下是一个修改后的脚本示例：

#!/usr/bin/env bash

if [ -f "./target/*.jar" ]; then
    # 执行 Jar 包
else
    echo "Jar file not found!"
fi
在 Git Bash 中，您可以使用 chmod 命令将脚本的权限改为 755：

chmod 755 path/to/your/script.sh
这样就可以在 Windows 上运行该脚本了。同时，请注意在 Windows 系统中路径分隔符为 \，而在 Shell 脚本中使用的是 /，因此需要对路径进行相应的转换。




2023/6/9 23:33:12

那插件配置<executable>/bin/bash</executable>，这里要改吗？

2023/6/9 23:33:46

是的，在 Git Bash 中，Bash 解释器的路径通常为 /usr/bin/bash，因此您需要将 Maven 插件配置中的 <executable> 标签改为该路径，如下所示：

<configuration>
    <executable>/usr/bin/bash</executable>
    <!-- 其他配置 -->
</configuration>
另外，请注意在 Windows 环境下，由于行分隔符不同，可能会导致一些脚本在 Git Bash 中无法正常运行。例如，在 Windows 中使用记事本编辑的脚本文件通常以 DOS 格式保存，而 Git Bash 只支持 UNIX 格式的文本文件。如果您遇到这种情况，请在 Git Bash 中打开文件并使用 dos2unix 命令将其转换为 UNIX 格式，例如：

dos2unix path/to/your/script.sh



2023/6/9 23:34:00

如果是bat脚本呢？

2023/6/9 23:34:20

如果您想使用 Windows 的批处理脚本（.bat 文件）来代替 Bash 脚本，则可以在 Maven 插件配置中指定 cmd 命令解释器的路径。

在 Windows 中，批处理文件是默认通过 cmd.exe 命令解释器来执行的。因此，您需要将 <executable> 标签改为 cmd.exe 的路径，并将参数中的命令以 /C 开头。例如：

<configuration>
    <executable>C:\Windows\System32\cmd.exe</executable>
    <arguments>
        <argument>/C</argument>
        <argument>path\to\your\script.bat</argument>
    </arguments>
    <!-- 其他配置 -->
</configuration>
在上面的配置中，我们假设您的 cmd.exe 命令解释器位于 C:\Windows\System32\ 目录下，并且您的批处理文件位于 path\to\your\ 目录下。

另外，请注意在 Windows 环境下，批处理文件的行结束符通常是 CRLF（回车换行），而 Git Bash 只支持 LF（换行）格式的文本文件。如果您遇到这种情况，请在 Git Bash 中打开文件并使用 unix2dos 命令将其转换为 CRLF 格式，例如：

unix2dos path\to\your\script.bat



2023/6/9 23:35:17

请提供示例bat脚本，判断jar文件存在则执行docker build后，执行docker run

2023/6/9 23:35:44

以下是一个示例的 Windows 批处理脚本，它会检查当前目录下是否存在名为 app.jar 的 JAR 文件。如果找到该文件，则会构建 Docker 镜像，并启动容器运行该镜像：

@echo off

set jarfile=app.jar
if not exist %jarfile% (
    echo Jar file "%jarfile%" not found!
    exit /b 1
)

set docker_image=my-docker-image
set docker_container=my-docker-container
set docker_port=8080

echo Building Docker image...
docker build -t %docker_image% .

echo Starting Docker container...
docker run -d -p %docker_port%:%docker_port% --name %docker_container% %docker_image%
在上述脚本中，我们首先检查当前目录是否存在名为 app.jar 的 JAR 文件。如果不存在，则输出错误消息并退出脚本。

接着，我们设置了几个变量，用于指定 Docker 镜像和容器的名称以及需要映射的端口号。然后，我们调用 docker build 命令来构建 Docker 镜像，并使用 -t 参数指定镜像的标签名。接下来，我们调用 docker run 命令来启动容器，并使用 -d 参数将容器设置为后台模式运行。我们还使用 -p 参数来映射容器内部的端口到主机上的相应端口，使得外部可以访问容器中运行的应用程序。最后，我们通过 --name 参数来指定容器的名称，以及使用之前构建的 Docker 镜像。

请注意，在 Windows 环境下，请使用 set 命令定义变量，而不是使用 Bash Shell 中的 export 命令。

您可以将上述脚本保存为名为 run.bat 的文件，并将其放置到包含 JAR 文件和 Dockerfile 的目录中。然后，在 Git Bash 中切换到该目录，执行以下命令即可运行脚本：

./run.bat


fabric8io插件把springboot打包成k8s镜像，部署到k8s

2023/6/11 18:26:54

fabric8io插件是一款非常方便的工具，可以帮助开发者将Spring Boot应用程序打包成Kubernetes镜像，并将其部署到Kubernetes集群中。

使用fabric8io插件，您可以通过以下步骤将Spring Boot应用程序打包成Kubernetes镜像并将其部署到Kubernetes集群中：

首先，在pom.xml文件中添加fabric8io插件的依赖关系：
<dependency>
    <groupId>io.fabric8</groupId>
    <artifactId>fabric8-maven-plugin</artifactId>
    <version>4.0.0</version>
</dependency>
接下来，在maven配置文件中添加fabric8io插件的配置信息，例如：
<plugin>
    <groupId>io.fabric8</groupId>
    <artifactId>fabric8-maven-plugin</artifactId>
    <version>4.0.0</version>
    <configuration>
        <dockerImage>${project.groupId}/${project.artifactId}:${project.version}</dockerImage>
        <generator>
            <config>
                <from>spring-boot</from>
            </config>
        </generator>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>build</goal>
                <goal>resource</goal>
            </goals>
        </execution>
    </executions>
</plugin>
最后，执行以下命令将应用程序构建成Kubernetes镜像并将其部署到Kubernetes集群：
mvn clean install fabric8:deploy
这将使用fabric8io插件构建Docker镜像、生成Kubernetes部署清单并将其应用于Kubernetes集群。

需要注意的是，您需要在执行此操作之前确保正确配置了Kubernetes集群，并且拥有访问该集群的权限。




2023/6/11 18:27:24

部署本地k8s需要yml

2023/6/11 18:27:37

是的，要在本地Kubernetes集群中部署应用程序，您需要使用YAML文件定义Kubernetes对象（例如Pod、Service、Deployment等）。这些YAML文件描述了Kubernetes集群中的对象和它们之间的关系。以下是一个示例YAML文件，它描述了一个简单的Deployment：

apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment
  labels:
    app: example-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: example-app
  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers:
        - name: example-container
          image: your-image-name
          ports:
            - containerPort: 8080
在这个示例中，YAML文件定义了一个名为"example-deployment"的Deployment对象，该对象包含三个Pod副本，并将它们标记为"app=example-app"。每个Pod包含一个名为"example-container"的容器，容器运行名为"your-image-name"的Docker镜像，并暴露端口号8080。

您可以使用kubectl命令行工具来创建和管理这些对象。例如，要创建上述Deployment对象，请将YAML文件保存为"example-deployment.yaml"并执行以下命令：

kubectl apply -f example-deployment.yaml
这将在您的Kubernetes集群中创建一个名为"example-deployment"的Deployment对象，该对象将自动创建三个Pod实例。




2023/6/11 18:28:10

fabric8io会自动识别该yml文件？

2023/6/11 18:28:20

fabric8io插件可以自动识别Kubernetes YAML文件，并根据这些文件创建或更新Kubernetes对象。在使用fabric8io插件时，只需将YAML文件放置在src/main/fabric8/kubernetes文件夹中即可。

例如，如果您有一个名为"example-deployment.yaml"的Kubernetes Deployment YAML文件，可以将其放置在src/main/fabric8/kubernetes文件夹中，然后执行以下命令：

mvn clean install fabric8:deploy
这将使用fabric8io插件将该YAML文件加载到Kubernetes集群中，并创建或更新Kubernetes Deployment对象。如果您想要手动创建或更新Kubernetes对象，也可以使用kubectl命令行工具来执行相应的操作。

需要注意的是，fabric8io插件提供了一些有用的特性，例如可以使用Maven属性和插值操作在YAML文件中设置变量值。最终的Kubernetes YAML文件将包含实际的变量值，这使得部署到不同环境的应用程序变得更加容易。