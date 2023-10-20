# Spring读取Resource

```java
File file = ResourceUtils.getFile("classpath:exceltmp/template_export.xls");
```

SpringBoot项目在Resource下的文件是存储在于jar这个文件里面，在磁盘上是没有真实路径存在的，它其实是位于jar内部的一个路径。所以通过ResourceUtils.getFile或者this.getClass().getResource("")方法无法正确获取文件。
有一种比较偷懒的做法：将文档放在项目外，应用可以读取到的一个固定目录。按正常的方式读取即可，但可维护性比较差，很容易被误操作丢失。

## 文本文件读取
这种情况下可以采用流的方式来读取文件，拿到文件流再进行相关的操作。如果你使用Spring框架的话，可以采用ClassPathResource来读取文件流，将文件读取成字符串才进行二次操作，比较适用于文本文件，如properties，txt，csv，SQL，json等，代码参考：

```java
String data = "";
ClassPathResource cpr = new ClassPathResource("static/file.txt");
try {
    byte[] bdata = FileCopyUtils.copyToByteArray(cpr.getInputStream());
    data = new String(bdata, StandardCharsets.UTF_8);
} catch (IOException e) {
    LOG.warn("IOException", e);
}

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.stream.Collectors;

import org.springframework.core.io.ClassPathResource;

public final class ClassPathResourceReader {
    /**
     * path:文件路径
     * @since JDK 1.8
     */
    private final String path;

    /**
     * content:文件内容
     * @since JDK 1.6
     */
    private String content;

    public ClassPathResourceReader(String path) {
        this.path = path;
    }

    public String getContent() {
        if (content == null) {
            try {
                ClassPathResource resource = new ClassPathResource(path);
                BufferedReader reader = new BufferedReader(new InputStreamReader(resource.getInputStream()));
                content = reader.lines().collect(Collectors.joining("\n"));
                reader.close();
            } catch (IOException ex) {
                throw new RuntimeException(ex);
            }
        }
        return content;
    }
}
```

## 非文本文件读取

更多的情况是读取非文本文件，比如xls，还是希望拿到一个文件，再去解析使用。参考代码如下：

```java
ClassPathResource classPathResource = new ClassPathResource("exceltmp/template_export.xls"");

InputStream inputStream = classPathResource.getInputStream();
//生成目标文件
File somethingFile = File.createTempFile("template_export_copy", ".xls");
try {
    FileUtils.copyInputStreamToFile(inputStream, somethingFile);
} finally {
    IOUtils.closeQuietly(inputStream);
}
```