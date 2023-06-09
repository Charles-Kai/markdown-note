2023/6/9 23:47:57

应用java把postman 某个collection有response example的请求，转化https://github.com/dreamhead/moco的json格式

2023/6/9 23:48:29

你可以使用Java中的第三方库，例如Jackson或Gson来将Postman Collection的响应示例转换为Moco JSON格式。

以下是示例代码：

java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.node.ObjectNode;
import java.io.File;
import java.io.IOException;
import java.nio.file.Files;

public class PostmanToMocoConverter {
    public static void main(String[] args) throws IOException {
        String postmanCollectionPath = "postman_collection.json";
        String mocoConfigPath = "moco_config.json";

        // 读取Postman Collection文件
        byte[] postmanCollectionBytes = Files.readAllBytes(new File(postmanCollectionPath).toPath());
        ObjectMapper objectMapper = new ObjectMapper();
        ObjectNode postmanCollection = (ObjectNode) objectMapper.readTree(postmanCollectionBytes);

        // 提取响应示例
        ObjectNode mocoConfig = objectMapper.createObjectNode();
        for (JsonNode item : postmanCollection.get("item")) {
            String itemName = item.get("name").asText();
            ObjectNode request = (ObjectNode) item.get("request");
            ObjectNode response = (ObjectNode) item.get("response");

            if (response != null && response.has("body")) {
                String responseBody = response.get("body").asText();
                if (!responseBody.isEmpty()) {
                    ObjectNode responseJson = (ObjectNode) objectMapper.readTree(responseBody);
                    mocoConfig.set(itemName, responseJson);
                }
            }
        }

        // 将Moco配置写入文件
        byte[] mocoConfigBytes = objectMapper.writerWithDefaultPrettyPrinter().writeValueAsBytes(mocoConfig);
        Files.write(new File(mocoConfigPath).toPath(), mocoConfigBytes);
    }
}
此代码将从名为postman_collection.json的文件中读取Postman Collection，在每个请求项目中查找响应示例，并将其转换为Moco JSON格式。最后，它将Moco配置写入名为moco_config.json的文件中。

请注意，此代码仅处理请求项目的直接子级。如果您的Postman Collection包含文件夹和子文件夹，请相应地修改代码以处理嵌套结构。




2023/6/9 23:49:52

转化逻辑必须包含请求的url和header，参数，以及返回的header和body

2023/6/9 23:50:33

以下是更新后的示例代码，该代码将请求的URL、headers、body以及响应的headers和body转换为Moco JSON格式：

java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.node.ObjectNode;

import java.util.HashMap;
import java.util.Map;

public class PostmanToMocoConverter {
    public static void main(String[] args) {
        String postmanCollectionPath = "postman_collection.json";
        String mocoConfigPath = "moco_config.json";

        ObjectMapper objectMapper = new ObjectMapper();
        ObjectNode mocoConfig = objectMapper.createObjectNode();

        try {
            // 读取Postman Collection文件
            PostmanCollection postmanCollection = objectMapper.readValue(new File(postmanCollectionPath), PostmanCollection.class);

            // 提取请求信息
            for (PostmanRequest request : postmanCollection.getItems()) {
                // 获取请求方法
                String method = request.getRequest().getMethod().toUpperCase();

                // 获取请求路径
                String path = request.getRequest().getUrl().getPath();

                // 获取请求头
                Map<String, String> headers = new HashMap<>();
                for (PostmanHeader header : request.getRequest().getHeader()) {
                    headers.put(header.getKey(), header.getValue());
                }

                // 获取请求体
                Object requestBody = null;
                if (request.getRequest().getBody() != null && request.getRequest().getBody().getRaw() != null) {
                    requestBody = objectMapper.readTree(request.getRequest().getBody().getRaw());
                }

                // 提取响应信息
                ObjectNode response = objectMapper.createObjectNode();
                if (request.getResponse() != null && request.getResponse().size() > 0) {
                    PostmanResponse firstResponse = request.getResponse().get(0);

                    // 获取响应状态码
                    int status = firstResponse.getCode();

                    // 获取响应头
                    Map<String, String> responseHeaders = new HashMap<>();
                    for (PostmanHeader header : firstResponse.getHeader()) {
                        responseHeaders.put(header.getKey(), header.getValue());
                    }

                    // 获取响应体
                    Object responseBody = objectMapper.readTree(firstResponse.getBody());
                    response.set("status", status);
                    response.set("headers", objectMapper.valueToTree(responseHeaders));
                    response.set("body", responseBody);
                }

                // 将请求信息和响应信息添加到Moco配置中
                ObjectNode requestNode = objectMapper.createObjectNode();
                requestNode.set("uri", objectMapper.valueToTree(path));
                if (!headers.isEmpty()) {
                    requestNode.set("headers", objectMapper.valueToTree(headers));
                }
                if (requestBody != null) {
                    requestNode.set("json", requestBody);
                }
                requestNode.set("response", response);

                mocoConfig.set(request.getName(), requestNode);
            }

            // 将Moco配置写入文件
            objectMapper.writerWithDefaultPrettyPrinter().writeValue(new File(mocoConfigPath), mocoConfig);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
这个新版本的代码将请求的URL、headers、body以及响应的headers和body提取为Moco所需的格式，并将它们放入一个ObjectNode对象中。然后，它将该对象添加到另一个ObjectNode对象中，并将整个对象写入名为moco_config.json的文件中。

请注意，此代码假定您已经定义了用于解析Postman Collection的PostmanCollection、PostmanRequest和PostmanHeader类。如果您还没有这些类，请根据Postman Collection的格式创建它们并使用Jackson或Gson等库来解析它们。




2023/6/9 23:51:10

请根据Postman Collection的格式创建定义了用于解析Postman Collection的PostmanCollection、PostmanRequest和PostmanHeader类。

2023/6/9 23:52:02

以下是定义了用于解析Postman Collection的PostmanCollection、PostmanRequest和PostmanHeader类的示例代码：

java
import java.util.List;

public class PostmanCollection {
    private List<PostmanRequest> item;

    public List<PostmanRequest> getItems() {
        return item;
    }

    public void setItems(List<PostmanRequest> item) {
        this.item = item;
    }
}

java
import java.util.List;

public class PostmanRequest {
    private String name;
    private PostmanRequestData request;
    private List<PostmanResponse> response;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public PostmanRequestData getRequest() {
        return request;
    }

    public void setRequest(PostmanRequestData request) {
        this.request = request;
    }

    public List<PostmanResponse> getResponse() {
        return response;
    }

    public void setResponse(List<PostmanResponse> response) {
        this.response = response;
    }
}

java
public class PostmanRequestData {
    private String method;
    private PostmanUrl url;
    private List<PostmanHeader> header;
    private PostmanRequestBody body;

    public String getMethod() {
        return method;
    }

    public void setMethod(String method) {
        this.method = method;
    }

    public PostmanUrl getUrl() {
        return url;
    }

    public void setUrl(PostmanUrl url) {
        this.url = url;
    }

    public List<PostmanHeader> getHeader() {
        return header;
    }

    public void setHeader(List<PostmanHeader> header) {
        this.header = header;
    }

    public PostmanRequestBody getBody() {
        return body;
    }

    public void setBody(PostmanRequestBody body) {
        this.body = body;
    }
}

java
public class PostmanUrl {
    private String raw;
    private List<PostmanQueryParam> query;

    public String getRaw() {
        return raw;
    }

    public void setRaw(String raw) {
        this.raw = raw;
    }

    public List<PostmanQueryParam> getQuery() {
        return query;
    }

    public void setQuery(List<PostmanQueryParam> query) {
        this.query = query;
    }

    public String getPath() {
        // 解析出请求路径
        String[] parts = raw.split("\\?");
        if (parts.length > 0) {
            return parts[0];
        } else {
            return "";
        }
    }
}

java
public class PostmanHeader {
    private String key;
    private String value;

    public String getKey() {
        return key;
    }

    public void setKey(String key) {
        this.key = key;
    }

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }
}

java
public class PostmanRequestBody {
    private String raw;

    public String getRaw() {
        return raw;
    }

    public void setRaw(String raw) {
        this.raw = raw;
    }
}

java
public class PostmanResponse {
    private int code;
    private List<PostmanHeader> header;
    private String body;

    public int getCode() {
        return code;
    }

    public void setCode(int code) {
        this.code = code;
    }

    public List<PostmanHeader> getHeader() {
        return header;
    }

    public void setHeader(List<PostmanHeader> header) {
        this.header = header;
    }

    public String getBody() {
        return body;
    }

    public void setBody(String body) {
        this.body = body;
    }
}

java
public class PostmanQueryParam {
    private String key;
    private String value;

    public String getKey() {
        return key;
    }

    public void setKey(String key) {
        this.key = key;
    }

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }
}

这些类的定义基于Postman Collection的JSON格式，其中包含名为item的顶级元素，每个元素都表示一个请求项目。每个请求项目都包含一个名为request的对象和一个名为response的数组，它们分别表示请求和响应。request对象包含一些基本信息，例如请求方法、URL、headers和body。response数组包含所有针对


2023/6/9 23:52:06

继续

2023/6/9 23:52:17

请求的响应，每个响应包含一个状态码、headers和body。

在这些类中，PostmanCollection类表示整个Postman Collection，其中包含一个名为item的列表，该列表由PostmanRequest对象组成。PostmanRequest类表示单个请求项目，包括请求的名称、请求信息和响应信息。PostmanRequestData类表示请求数据，包括HTTP方法、URL、headers和body。PostmanUrl类表示请求的URL，包括路径和查询参数。PostmanHeader类表示HTTP header，包括键和值。PostmanRequestBody类表示请求体的原始内容。PostmanResponse类表示请求的响应，包括状态码、headers和body。最后，PostmanQueryParam类表示查询参数，包括键和值。

用json-path重新实现上面的objectmapper方案

2023/6/9 23:58:25

以下是使用json-path库重新实现将Postman Collection转换为Moco JSON格式的示例代码：

java
import com.jayway.jsonpath.Configuration;
import com.jayway.jsonpath.JsonPath;
import com.jayway.jsonpath.Option;
import com.jayway.jsonpath.TypeRef;

import java.io.File;
import java.io.IOException;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class PostmanToMocoConverter {
    public static void main(String[] args) {
        String postmanCollectionPath = "postman_collection.json";
        String mocoConfigPath = "moco_config.json";

        Configuration configuration = Configuration.builder()
                .options(Option.SUPPRESS_EXCEPTIONS)
                .build();

        try {
            // 读取Postman Collection文件
            String json = FileUtils.readFileToString(new File(postmanCollectionPath));

            // 提取请求信息
            List<Map<String, Object>> items = JsonPath.using(configuration).parse(json).read("$.item", new TypeRef<List<Map<String, Object>>>() {});

            Map<String, Map<String, Object>> mocoConfig = new HashMap<>();

            for (Map<String, Object> item : items) {
                Map<String, Object> request = (Map<String, Object>) item.get("request");

                // 获取请求方法
                String method = (String) request.get("method");

                // 获取请求路径
                String path = JsonPath.using(configuration).parse(request).read("$.url.path");

                // 获取请求头
                Map<String, String> headers = new HashMap<>();
                List<Map<String, String>> headerList = JsonPath.using(configuration).parse(request).read("$.header[*]");
                for (Map<String, String> header : headerList) {
                    headers.put(header.get("key"), header.get("value"));
                }

                // 获取请求体
                Object requestBody = null;
                String rawBody = JsonPath.using(configuration).parse(request).read("$.body.raw");
                if (rawBody != null && !rawBody.isEmpty()) {
                    requestBody = configuration.jsonProvider().parse(rawBody);
                }

                // 提取响应信息
                List<Map<String, Object>> responseList = (List<Map<String, Object>>) item.get("response");

                Map<String, Object> response = new HashMap<>();
                if (responseList != null && responseList.size() > 0) {
                    Map<String, Object> firstResponse = responseList.get(0);

                    // 获取响应状态码
                    int status = (int) firstResponse.get("code");

                    // 获取响应头
                    Map<String, String> responseHeaders = new HashMap<>();
                    List<Map<String, String>> responseHeaderList = JsonPath.using(configuration).parse(firstResponse).read("$.header[*]");
                    for (Map<String, String> header : responseHeaderList) {
                        responseHeaders.put(header.get("key"), header.get("value"));
                    }

                    // 获取响应体
                    Object responseBody = configuration.jsonProvider().parse((String) firstResponse.get("body"));

                    response.put("status", status);
                    response.put("headers", responseHeaders);
                    response.put("body", responseBody);
                }

                // 将请求信息和响应信息添加到Moco配置中
                Map<String, Object> requestNode = new HashMap<>();
                requestNode.put("uri", path);
                if (!headers.isEmpty()) {
                    requestNode.put("headers", headers);
                }
                if (requestBody != null) {
                    requestNode.put("json", requestBody);
                }
                requestNode.put("response", response);

                mocoConfig.put((String) item.get("name"), requestNode);
            }

            // 将Moco配置写入文件
            String mocoConfigJson = configuration.jsonProvider().toJson(mocoConfig);
            FileUtils.writeStringToFile(new File(mocoConfigPath), mocoConfigJson, "UTF-8");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
这个新版本的代码使用了json-path库来解析Postman Collection中的JSON数据。它可以轻松地提取请求的URL、headers、body以及响应的headers和body，然后将它们转换为Moco JSON格式。

请注意，在此示例中，我们使用了Map对象来表示Moco配置。与前一个示例不同，这里没有使用Jackson或Gson等对象映射库。而是使用json-path库的Configuration和jsonProvider()方法来创建和操作JSON对象。