如何用com.sun.tools.javac的方式读取下面代码，获取import的类名：
package chapter1;

import java.net.MalformedURLException;
import java.net.URL;

public class TestJavac {
    private URL url;

    public TestJavac(String urlStr) throws MalformedURLException {
        this.url = new URL(urlStr);
    }
}

2023/6/19 22:51:59

使用com.sun.tools.javac编译器提供的API，可以通过以下代码获取TestJavac类的import语句中所引入的类的类名：

java
import com.sun.tools.javac.api.*;
import com.sun.tools.javac.tree.JCTree.*;

import java.io.IOException;
import java.io.StringWriter;
import java.util.ArrayList;
import java.util.List;

public class ImportScanner {
    public static void main(String[] args) throws IOException {
        String code = "package chapter1;\n" +
                "\n" +
                "import java.net.MalformedURLException;\n" +
                "import java.net.URL;\n" +
                "\n" +
                "public class TestJavac {\n" +
                "    private URL url;\n" +
                "\n" +
                "    public TestJavac(String urlStr) throws MalformedURLException {\n" +
                "        this.url = new URL(urlStr);\n" +
                "    }\n" +
                "}\n";

        // 初始化编译器实例
        JavacTool tool = JavacTool.create();
        // 获取JavaFileObject实例
        JavaFileObject fileObject = new SimpleJavaFileObject(
                URI.create("file:///TestJavac.java"),
                JavaFileObject.Kind.SOURCE) {
            @Override
            public CharSequence getCharContent(boolean ignoreEncodingErrors) throws IOException {
                return code;
            }
        };
        // 创建编译任务
        List<JavaFileObject> files = new ArrayList<>();
        files.add(fileObject);
        JavacTask task = tool.getTask(null, null, null, null, null, files);
        // 执行编译任务并获取AST树
        Iterable<? extends CompilationUnitTree> trees = task.parse();
        for (CompilationUnitTree tree : trees) {
            // 获取导入的类名列表
            List<JCTree.JCImport> imports = tree.getImports();
            for (JCTree.JCImport jcImport : imports) {
                String importName = jcImport.qualid.toString();
                System.out.println(importName);
            }
        }
    }
}
上述代码中，我们首先定义了一个字符串变量code，其中包含了我们要读取的源代码。然后，通过创建SimpleJavaFileObject实例来将该源代码转换为JavaFileObject实例。接着，我们使用JavacTool.create()方法创建编译器实例，并使用其getTask()方法创建编译任务。执行parse()方法可以得到AST树，从而获取import语句，并通过qualid.toString()方法获取所导入的类的类名。在本例中，输出结果为：

java.net.MalformedURLException
java.net.URL