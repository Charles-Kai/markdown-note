Java MavenXpp3Writer类代码示例
https://www.cnblogs.com/996--icu/p/13603281.html
Java MavenXpp3Writer类代码示例
https://vimsky.com/examples/detail/java-class-org.apache.maven.model.io.xpp3.MavenXpp3Writer.html
API docs:
https://maven.apache.org/ref/3.5.0/maven-model/apidocs/org/apache/maven/model/io/xpp3/MavenXpp3Writer.html


To preserve the order of elements in the pom.xml file while using MavenXpp3Reader/MavenXpp3Writer, you can set the modelEncoding property to "UTF-8" while reading the pom.xml file and writing it back to the file. This will ensure that the order of elements in the pom.xml file is preserved.

Here's an example code snippet in Java that demonstrates how to do this:

In this example, we first read the pom.xml file using MavenXpp3Reader and set the modelEncoding property to "UTF-8". We then modify the Model object as needed and write it back to the pom.xml file using MavenXpp3Writer. The order of elements in the pom.xml file will be preserved. 
不是乱码，是MavenXpp3Writer会丢失读取的pom文件的结构和注释
When using MavenXpp3Writer to write a pom.xml file, it may lose the structure and comments of the original file. This is because MavenXpp3Writer only writes the model data to the file and does not preserve the original formatting and comments.

To preserve the structure and comments of the original pom.xml file, you can use XmlStreamWriter instead of MavenXpp3Writer. Here's an example code snippet in Java that demonstrates how to do this:

In this example, we first read the pom.xml file using MavenXpp3Reader. We then modify the Model object as needed and write it back to the pom.xml file using XmlStreamWriter and MavenXpp3Writer. The XmlStreamWriter is used to preserve the original formatting and comments of the pom.xml file. The order of elements in the pom.xml file will also be preserved. 

import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.util.Properties;
import java.util.Set;
import java.util.TreeMap;
import java.util.LinkedHashSet;

import org.apache.maven.model.Model;
import org.apache.maven.model.io.xpp3.MavenXpp3Reader;
import org.apache.maven.model.io.xpp3.MavenXpp3ReaderEx;
import org.apache.maven.model.io.xpp3.MavenXpp3Writer;

import groovy.lang.Grab;
import groovy.lang.GrabResolver;

@GrabResolver(name = "mavenCentral", root = "https://repo1.maven.org/maven2/")
@Grab(group = "org.apache.maven", module = "maven-model", version = "3.3.9")

public class Main {
    public static void main(String[] args) throws Exception {
        File f = new File(args[0]);
        MavenXpp3ReaderEx reader = new MavenXpp3ReaderEx();
        Model model = reader.read(new FileInputStream(f), true, null);

        // your modifications
        model.getProperties().setProperty("plexusVersion","99.99");

        // reorder properties keys by line number
        Model.InputLocationTracker locations = model.getLocationTracker().getLocation( "properties" );

        TreeMap<Integer, String> mapByLine = new TreeMap<Integer, String>();

        Properties props = model.getProperties();
        Set<Object> keys = props.keySet();

        for (Object key : keys) {
            Object val = props.get(key);
            Model.InputLocation keyLoc = locations.getLocation((String)key);
            mapByLine.put(keyLoc.getLineNumber(), (String)key);
        }

        Properties newProps = new Properties() {
            public Set<Object> keySet() {
                return new LinkedHashSet<Object>(mapByLine.values());
            }
        };

        newProps.putAll(model.getProperties());

        model.setProperties(newProps);

        MavenXpp3Writer writer = new MavenXpp3Writer();
        File fout = new File("output-pom.xml");
        writer.write(new FileOutputStream(fout), model);
    }
}
