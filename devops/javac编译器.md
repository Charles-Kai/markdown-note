如果你想使用Java源代码分析来获取方法的局部变量调用的方法以及该方法的参数类型和名称，你可以使用JavaParser这个开源库。JavaParser可以将Java源代码解析为抽象语法树（AST），并且可以让你遍历和修改AST。

以下是一个使用JavaParser的例子，它可以获取到方法的局部变量调用的方法以及该方法的参数类型和名称：
import com.github.javaparser.JavaParser;
import com.github.javaparser.ast.CompilationUnit;
import com.github.javaparser.ast.body.MethodDeclaration;
import com.github.javaparser.ast.body.Parameter;
import com.github.javaparser.ast.expr.MethodCallExpr;
import com.github.javaparser.ast.visitor.VoidVisitorAdapter;

public class MethodAnalyzer {
    public static void analyze(String sourceCode, String methodName) {
        CompilationUnit cu = JavaParser.parse(sourceCode);

        new MethodVisitor(methodName).visit(cu, null);
    }

    private static class MethodVisitor extends VoidVisitorAdapter<Void> {
        private final String methodName;

        public MethodVisitor(String methodName) {
            this.methodName = methodName;
        }

        @Override
        public void visit(MethodDeclaration md, Void arg) {
            if (md.getNameAsString().equals(methodName)) {
                md.accept(new MethodCallVisitor(), null);
            }
        }
    }

    private static class MethodCallVisitor extends VoidVisitorAdapter<Void> {
        @Override
        public void visit(MethodCallExpr mce, Void arg) {
            if (mce.getNameAsString().equals("get")) {
                for (com.github.javaparser.ast.type.Type type : mce.getArgumentTypes()) {
                    System.out.println("Method 'get' is called with parameter type: " + type);
                }
            }
        }
    }
}

在这个例子中，我们首先使用JavaParser将Java源代码解析为抽象语法树（AST）。然后，我们创建了一个MethodVisitor，它可以访问所有的方法声明，并且检查方法的名称是否与我们要查找的方法名称相同。如果方法的名称相同，那么我们就创建了一个MethodCallVisitor，它可以访问方法内部的所有方法调用。然后，我们检查每个方法调用的名称是否为get，如果是，那么我们就打印出方法调用的参数类型。

请注意，这个例子只能获取到方法调用的参数类型，不能获取到参数的名称。如果你需要获取参数的名称，你可能需要使用其他技术，如反射API或者字节码操作库。

总的来说，使用Java源代码分析来获取方法的局部变量调用的方法以及该方法的参数类型和名称是一项复杂的任务。你需要对Java语法和语义有深入的理解，并且可能需要使用JavaParser或其他技术。 


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