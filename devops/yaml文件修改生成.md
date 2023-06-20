在Java中，我们可以使用SnakeYAML或Jackson库来读取和修改YAML文件。以下是使用这两个库的示例。

## 使用SnakeYAML

首先，你需要将SnakeYAML库添加到你的项目中。如果你使用的是Maven，可以在pom.xml文件中添加以下依赖：
<dependency>
    <groupId>org.yaml</groupId>
    <artifactId>snakeyaml</artifactId>
    <version>1.28</version>
</dependency>

然后，你可以使用以下代码来读取和修改YAML文件：
import org.yaml.snakeyaml.Yaml;

import java.io.FileInputStream;
import java.io.FileWriter;
import java.io.InputStream;
import java.util.Map;

public class YamlExample {
    public static void main(String[] args) throws Exception {
        Yaml yaml = new Yaml();
        InputStream inputStream = new FileInputStream("/path/to/your/file.yaml");
        Map<String, Object> obj = yaml.load(inputStream);

        // 修改内容
        obj.put("newKey", "newValue");

        // 写回文件
        FileWriter writer = new FileWriter("/path/to/your/file.yaml");
        yaml.dump(obj, writer);
    }
}

## 使用Jackson

首先，你需要将Jackson库添加到你的项目中。如果你使用的是Maven，可以在pom.xml文件中添加以下依赖：
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-yaml</artifactId>
    <version>2.12.3</version>
</dependency>

然后，你可以使用以下代码来读取和修改YAML文件：
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.dataformat.yaml.YAMLFactory;

import java.io.File;
import java.util.Map;

public class YamlExample {
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper(new YAMLFactory());
        File yamlFile = new File("/path/to/your/file.yaml");

        Map<String, Object> obj = mapper.readValue(yamlFile, Map.class);

        // 修改内容
        obj.put("newKey", "newValue");

        // 写回文件
        mapper.writeValue(yamlFile, obj);
    }
}

请注意，这两个示例都假设你的YAML文件包含一个顶级的映射。如果你的YAML文件的结构不同，你可能需要调整这些示例以适应你的具体情况。 
对路径比较深的属性，考虑易用性，哪个比较好
对于路径较深的属性，我建议使用Jackson库。Jackson提供了一种更灵活的方式来处理嵌套的属性。你可以使用JsonNode或者ObjectNode来访问和修改嵌套的属性，而不需要知道整个对象的完整结构。

以下是一个例子，展示了如何使用Jackson来修改嵌套的属性：
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.node.ObjectNode;
import com.fasterxml.jackson.dataformat.yaml.YAMLFactory;

import java.io.File;

public class YamlExample {
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper(new YAMLFactory());
        File yamlFile = new File("/path/to/your/file.yaml");

        ObjectNode root = (ObjectNode) mapper.readTree(yamlFile);

        // 修改嵌套的属性
        root.with("parent").put("child", "newValue");

        // 写回文件
        mapper.writeValue(yamlFile, root);
    }
}

在这个例子中，with方法用于获取或创建一个子节点。如果该节点不存在，with方法将创建一个新的ObjectNode。然后，你可以使用put方法来修改该节点的属性。

这种方式的优点是，你不需要知道整个对象的完整结构，只需要知道你想要修改的属性的路径。这使得处理复杂的YAML文件变得更加容易。 
