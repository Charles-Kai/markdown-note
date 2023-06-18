wiremock 

wiremock reload mappings

2023/6/18 23:14:57

在 WireMock 中重新加载映射可以通过以下步骤实现：

通过 HTTP 协议向 WireMock 发送一个 POST 请求到 /__admin/mappings/reset 路径，如下所示：
POST /__admin/mappings/reset HTTP/1.1
Host: localhost:8080
WireMock 将重新加载 mappings 目录下的所有映射文件，并清除所有已经匹配过的请求。
请注意，这可能会影响正在进行的测试，因此应该谨慎使用。另外，如果你想要更加细致的控制映射文件的加载，可以考虑使用 WireMock 的 Java API 来动态地添加、删除、更新映射。





如果不在__files文件夹下，在上级文件夹呢？

2023/6/18 23:25:08

如果您要在WireMock的静态文件配置中返回不在__files目录下的JSON文件，则可以使用相对路径来指定该文件的位置。假设您的JSON文件位于与WireMock静态文件配置相同的根目录下的json文件夹中，您可以使用以下方式指定该文件的位置：

json
{
  "mappings": [
    {
      "request": {
        "method": "GET",
        "urlPattern": "/example"
      },
      "response": {
        "status": 200,
        "headers": {
          "Content-Type": "application/json"
        },
        "bodyFileName": "../json/your_file.json"
      }
    }
  ]
}
在这个例子中，..表示上一级目录，然后我们进入了json目录，并指定了我们要返回的JSON文件的名称：your_file.json。

请注意，您需要在HTTP响应中设置正确的Content-Type头以确保客户端正确地解析响应。在这个例子中，我们将Content-Type设置为application/json，因为响应是JSON格式的。

同时，您还需要在WireMock启动时指定静态文件配置文件的路径。例如，在命令行模式下，您可以使用如下命令运行WireMock：

java -jar wiremock-standalone-[version].jar --port [port] --root-dir /path/to/configuration/files
其中，--root-dir参数用于指定包含WireMock静态文件配置的根目录路径。