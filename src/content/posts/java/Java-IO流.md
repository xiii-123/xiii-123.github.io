---
title: Java-IO流
published: 2025-03-14
description: 'Java中IO流的常见用法'
image: ''
tags: [java, IO]
category: 'java'
draft: false 
---
# IO流的继承结构
## 字节流
```
java.io.InputStream
    ├── java.io.ByteArrayInputStream
    ├── java.io.BufferedInputStream
    ├── java.io.DataInputStream
    ├── java.io.FileInputStream
    ├── java.io.FilterInputStream
    │   ├── java.io.PushbackInputStream
    ├── java.io.ObjectInputStream
    ├── java.io.PipedInputStream
java.io.OutputStream
    ├── java.io.ByteArrayOutputStream
    ├── java.io.BufferedOutputStream
    ├── java.io.DataOutputStream
    ├── java.io.FileOutputStream
    ├── java.io.FilterOutputStream
    │   ├── java.io.PrintStream
    ├── java.io.ObjectOutputStream
    ├── java.io.PipedOutputStream
```
## 字符流
```
java.io.Reader
    ├── java.io.BufferedReader
    ├── java.io.CharArrayReader
    ├── java.io.FilterReader
    ├── java.io.InputStreamReader
    ├── java.io.PipedReader
    ├── java.io.StringReader
java.io.Writer
    ├── java.io.BufferedWriter
    ├── java.io.CharArrayWriter
    ├── java.io.FilterWriter
    ├── java.io.OutputStreamWriter
    ├── java.io.PipedWriter
    ├── java.io.StringWriter
```
## 带Buffer的流
### 字节输入流 BufferedInputStream
继承结构：
```
java.io.InputStream
    ├── java.io.FilterInputStream
        ├── java.io.BufferedInputStream
```
常用方法：
- `public int read()`：从输入流中读取数据的下一个字节。
- `public int read(byte[] b, int off, int len)`：从输入流中读取最多`len`个字节的数据到一个字节数组中。
- `public void close()`：关闭输入流并释放与流相关的系统资源。
代码示例：
```java
import java.io.BufferedInputStream;
import java.io.FileInputStream;
import java.io.IOException;
public class BufferedInputStreamExample {
    public static void main(String[] args) {
        try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream("example.txt"))) {
            int data;
            while ((data = bis.read()) != -1) {
                System.out.print((char) data);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
### 字节输出流 BufferedOutputStream
继承结构：
```
java.io.OutputStream
    ├── java.io.FilterOutputStream
        ├── java.io.BufferedOutputStream
```
常用方法：
- `public void write(int b)`：写入一个字节到输出流。
- `public void write(byte[] b, int off, int len)`：写入字节数组的一部分到输出流。
- `public void flush()`：刷新输出流并强制写出所有缓冲的输出字节。
代码示例：
```java
import java.io.BufferedOutputStream;
import java.io.FileOutputStream;
import java.io.IOException;
public class BufferedOutputStreamExample {
    public static void main(String[] args) {
        try (BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("example.txt"))) {
            String data = "Hello, World!";
            byte[] bytes = data.getBytes();
            bos.write(bytes);
            bos.flush();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
### 字符输入流 BufferedReader
继承结构：
```
java.io.Reader
    ├── java.io.BufferedReader
```
常用方法：
- `public int read()`：读取单个字符。
- `public int read(char[] cbuf, int off, int len)`：将字符读入数组的一部分。
- `public String readLine()`：读取一个文本行。
代码示例：
```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
public class BufferedReaderExample {
    public static void main(String[] args) {
        try (BufferedReader br = new BufferedReader(new FileReader("example.txt"))) {
            String line;
            while ((line = br.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
### 字符输出流 BufferedWriter
继承结构：
```
java.io.Writer
    ├── java.io.BufferedWriter
```
常用方法：
- `public void write(int c)`：写入单个字符。
- `public void write(char[] cbuf, int off, int len)`：写入字符数组的一部分。
- `public void write(String str, int off, int len)`：写入字符串的一部分。
- `public void newLine()`：写入一个行分隔符。
代码示例：
```java
import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.IOException;
public class BufferedWriterExample {
    public static void main(String[] args) {
        try (BufferedWriter bw = new BufferedWriter(new FileWriter("example.txt"))) {
            bw.write("Hello, World!");
            bw.newLine();
            bw.write("This is a buffered writer example.");
            bw.flush();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## 转换流
所谓转换流，就是将字节流转换为字符流，通过以下是两个类实现：
- `InputStreamReader extends Reader`
- `OutputStreamWriter extends Writer`

为什么要将字节流转换为字符流呢？通常有以下两种情况：
1. 指定字符集读写数据（JDK11之后淘汰）
2. 字节流想要使用字符流中的方法（例如从网络流中读取文本）

让我们来看三段示例代码：
### 1. 使用 InputStreamReader 和 OutputStreamWriter 从 GBK 编码的文件中读取文件，并写成 UTF-8
```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.io.IOException;
public class ConvertEncoding {
    public static void main(String[] args) {
        String inputPath = "input.txt"; // GBK 编码的文件
        String outputPath = "output.txt"; // 将要写入的 UTF-8 文件
        try (InputStreamReader isr = new InputStreamReader(new FileInputStream(inputPath), "GBK");
             OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream(outputPath), "UTF-8")) {
            char[] buffer = new char[1024];
            int length;
            while ((length = isr.read(buffer)) != -1) {
                osw.write(buffer, 0, length);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
### 2. 使用 FileReader 和 FileWriter 简化上述功能
请注意，FileReader 和 FileWriter 使用平台默认的字符编码，这通常不是 GBK 或 UTF-8。以下代码仅作为示例，实际上可能不会正确处理非默认编码的文件。
```java
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
public class SimplifiedConvert {
    public static void main(String[] args) {
        String inputPath = "input.txt"; // 假设文件已经是平台默认编码
        String outputPath = "output.txt"; // 将以平台默认编码写入
        try (FileReader fr = new FileReader(inputPath);
             FileWriter fw = new FileWriter(outputPath)) {
            char[] buffer = new char[1024];
            int length;
            while ((length = fr.read(buffer)) != -1) {
                fw.write(buffer, 0, length);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
### 3. 将 InputStreamReader 包装成 BufferedReader 并按行读取文本
```java
import java.io.FileInputStream;
import java.io.InputStreamReader;
import java.io.BufferedReader;
import java.io.IOException;
public class BufferedReadLine {
    public static void main(String[] args) {
        String path = "input.txt"; // 文件路径
        String charset = "GBK"; // 文件编码
        try (InputStreamReader isr = new InputStreamReader(new FileInputStream(path), charset);
             BufferedReader br = new BufferedReader(isr)) {
            String line;
            while ((line = br.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
在第三个示例中，我们使用 InputStreamReader 来处理文件编码，并将其包装在 BufferedReader 中以按行读取文本。这样可以有效地读取文本文件，同时处理特定的字符编码。

## 序列化流

序列化流在 Java 中是指将对象状态转换为可存储或可传输的形式（即字节流）的过程。这种字节流可以被保存到文件中、在网络中传输，或者存储在数据库中。反序列化是序列化的逆过程，它将字节流重新转换成对象。
Java 提供了 `ObjectOutputStream` 类用于对象的序列化，以及 `ObjectInputStream` 类用于对象的反序列化。
### 序列化的条件
- 类必须实现 `java.io.Serializable` 接口或者 `java.io.Externalizable` 接口。
- 类的所有属性必须是可序列化的。如果某个属性不需要被序列化，可以使用 `transient` 关键字修饰。
### 示例代码
以下是一个简单的示例，演示如何使用 `ObjectOutputStream` 和 `ObjectInputStream` 进行对象的序列化和反序列化。
#### 定义一个可序列化的类
```java
import java.io.Serializable;
public class Person implements Serializable {
    private String name;
    private int age;
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    // getter 和 setter 方法
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    @Override
    public String toString() {
        return "Person{name='" + name + "', age=" + age + '}';
    }
}
```
#### 序列化对象
```java
import java.io.FileOutputStream;
import java.io.ObjectOutputStream;
public class SerializationDemo {
    public static void main(String[] args) {
        Person person = new Person("Alice", 30);
        try (FileOutputStream fileOut = new FileOutputStream("person.ser");
             ObjectOutputStream out = new ObjectOutputStream(fileOut)) {
            out.writeObject(person);
            System.out.println("Object has been serialized.");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
#### 反序列化对象
```java
import java.io.FileInputStream;
import java.io.ObjectInputStream;
public class DeserializationDemo {
    public static void main(String[] args) {
        Person person = null;
        try (FileInputStream fileIn = new FileInputStream("person.ser");
             ObjectInputStream in = new ObjectInputStream(fileIn)) {
            person = (Person) in.readObject();
            System.out.println("Object has been deserialized.");
        } catch (Exception e) {
            e.printStackTrace();
        }
        System.out.println(person);
    }
}
```
在上述示例中，我们首先创建了一个 `Person` 对象，并使用 `ObjectOutputStream` 将其序列化到文件 `person.ser` 中。然后，我们使用 `ObjectInputStream` 从该文件中读取数据，并反序列化回 `Person` 对象。
需要注意的是，序列化 ID (`serialVersionUID`) 是非常重要的，它用于验证序列化的对象和类定义是否兼容。如果不指定，Java 编译器会根据类的详细信息自动生成一个，但这可能会导致不兼容问题，特别是在类的定义发生变化后。因此，推荐显式地定义 `serialVersionUID`。

### 序列化对象ID
当我们在序列化一个对象之后，将类修改（例如添加成员变量）再反序列化该对象时，就会出现一个问题：
```bash
java.io.InvalidClassException: Person; local class incompatible: stream classdesc serialVersionUID = 3093225108327135571, local class serialVersionUID = -7140855319140760877
```
原因是Java在序列化时，会给类生成一个`serialVersionUID`，当反序列化时，检测到该ID不匹配就会报错。为了解决这个问题，我们可以打开IDEA的设置，让IDEA帮我们生成一个`serialVersionUID`，并且在修改类时保持此ID不变即可。
需要勾选IDEA的两个设置选项：
- Serializablenon-static inner class without 'serialVersionUID'
- Transient field in non-serializable class

## 打印流
打印流 `PrintStream` 和 `PrintWriter` 是 Java I/O 包中用于方便地打印基本数据类型（如 int、float 等）和对象的方法。它们都提供了丰富的重载方法，使得数据打印变得简单。
### PrintStream 特点
- `PrintStream` 是一个字节输出流，它继承了 `FilterOutputStream` 类。
- 它可以直接打印字符串和基本数据类型的值，不需要手动转换。
- 它提供了自动行刷新功能，即当调用 `println` 方法时，会自动调用 `flush` 方法。
- 它通常用于输出到控制台，例如 `System.out` 就是一个 `PrintStream` 对象。
- 默认情况下，`PrintStream` 不会抛出 IOException。
### PrintWriter 特点
- `PrintWriter` 是一个字符输出流，它继承了 `Writer` 类。
- 它同样可以直接打印字符串和基本数据类型的值。
- `PrintWriter` 可以指定是否自动行刷新，通过构造函数的布尔参数 `autoFlush` 来控制。
- 它可以抛出 IOException，也可以不抛出，取决于构造函数的参数。
- `PrintWriter` 提供了更多的构造函数，可以接受不同的输出目的地，如文件、输出流等。
### PrintStream 示例代码
下面是一个使用 `PrintStream` 的示例，它将文本写入文件：
```java
import java.io.FileOutputStream;
import java.io.PrintStream;
public class PrintStreamExample {
    public static void main(String[] args) {
        try (FileOutputStream fileOut = new FileOutputStream("example.txt");
             PrintStream printStream = new PrintStream(fileOut)) {
            
            printStream.println("Hello, World!");
            printStream.println(12345);
            printStream.println(true);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
### PrintWriter 示例代码
下面是一个使用 `PrintWriter` 的示例，它同样将文本写入文件，并启用自动行刷新：
```java
import java.io.FileWriter;
import java.io.PrintWriter;
public class PrintWriterExample {
    public static void main(String[] args) {
        try (FileWriter fileWriter = new FileWriter("example.txt");
             PrintWriter printWriter = new PrintWriter(fileWriter, true)) {
            
            printWriter.println("Hello, World!");
            printWriter.println(12345);
            printWriter.println(false);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
让我们看看结果是不是所见即所得：
```bash
Hello, World!
12345
false
```

## 压缩流和解压缩流

压缩流（Compression Streams）和解压缩流（Decompression Streams）是用于处理数据压缩和解压缩的流。这些流可以用于减少数据大小，以便更有效地存储或传输数据。以下是Java中常用的压缩流和解压缩流：
### 压缩流
1. **GZIPOutputStream**: 用于压缩数据，以GZIP格式输出。
2. **ZipOutputStream**: 用于将数据压缩成ZIP格式。
### 解压缩流
1. **GZIPInputStream**: 用于解压缩GZIP格式的数据。
2. **ZipInputStream**: 用于解压缩ZIP格式的数据。
### 常用方法
#### GZIPOutputStream 和 GZIPInputStream
- `public GZIPOutputStream(OutputStream out)`：创建一个GZIPOutputStream实例，用于压缩数据并写入指定的输出流。
- `public GZIPInputStream(InputStream in)`：创建一个GZIPInputStream实例，用于从指定的输入流读取GZIP压缩数据。
#### ZipOutputStream 和 ZipInputStream
- `public ZipOutputStream(OutputStream out)`：创建一个ZipOutputStream实例，用于将数据压缩成ZIP格式并写入指定的输出流。
- `public ZipInputStream(InputStream in)`：创建一个ZipInputStream实例，用于从指定的输入流读取ZIP压缩数据。
### 示例代码
以下是一个简单的示例，演示如何使用`GZIPOutputStream`和`GZIPInputStream`进行压缩和解压缩字符串数据。
#### 压缩字符串
```java
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.util.zip.GZIPOutputStream;
public class CompressionExample {
    public static byte[] compressString(String str) throws IOException {
        if (str == null || str.length() == 0) {
            return null;
        }
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        GZIPOutputStream gzipOutputStream = new GZIPOutputStream(byteArrayOutputStream);
        gzipOutputStream.write(str.getBytes());
        gzipOutputStream.close();
        return byteArrayOutputStream.toByteArray();
    }
    public static void main(String[] args) {
        try {
            String originalString = "This is a string that we want to compress.";
            byte[] compressedString = compressString(originalString);
            System.out.println("Original String length: " + originalString.length());
            System.out.println("Compressed String length: " + compressedString.length);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
#### 解压缩字符串
```java
import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.util.zip.GZIPInputStream;
public class DecompressionExample {
    public static String decompressString(byte[] compressedBytes) throws IOException {
        if (compressedBytes == null || compressedBytes.length == 0) {
            return null;
        }
        ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(compressedBytes);
        GZIPInputStream gzipInputStream = new GZIPInputStream(byteArrayInputStream);
        StringBuilder outStr = new StringBuilder();
        byte[] buffer = new byte[1024];
        int len;
        while ((len = gzipInputStream.read(buffer)) > 0) {
            outStr.append(new String(buffer, 0, len));
        }
        gzipInputStream.close();
        return outStr.toString();
    }
    public static void main(String[] args) {
        try {
            byte[] compressedString = ...; // byte array from the compression example
            String decompressedString = decompressString(compressedString);
            System.out.println("Decompressed String: " + decompressedString);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
## Commons-io
Apache Commons IO 是一个用于扩展和增强 Java IO 类的实用程序库。它提供了一系列用于处理文件、流、字节和字符的类和方法。以下是 Commons IO 包中按类型分类的一些常见方法：
### 文件操作
- `FileUtils`
  - `copyFile(File srcFile, File destFile)`: 复制文件。
  - `deleteDirectory(File directory)`: 删除目录及其所有内容。
  - `forceDelete(File file)`: 强制删除文件或目录。
  - `writeStringToFile(File file, String data, String encoding)`: 将字符串写入文件。
  - `readFileToString(File file, String encoding)`: 从文件读取字符串。
  - `listFiles(File directory, String[] extensions, boolean recursive)`: 列出目录中的文件，可选地包含特定扩展名和递归子目录。
### 输入/输出流操作
- `IOUtils`
  - `copy(InputStream input, OutputStream output)`: 从输入流复制到输出流。
  - `toByteArray(InputStream input)`: 从输入流读取所有字节。
  - `toString(InputStream input, String encoding)`: 从输入流读取字符串。
  - `write(byte[] data, OutputStream output)`: 将字节数组写入输出流。
  - `closeQuietly(Closeable closeable)`: 关闭可关闭对象，不抛出异常。
### 文件监视
- `FileAlterationObserver`
  - 用于监视文件系统中的目录和文件的变化。
### 文件比较
- `FileComparator`
  - 用于比较文件的不同属性，如名称、大小、最后修改时间等。
### 文件过滤器
- `IOFileFilter`
  - 用于过滤文件和目录，例如 `FileFilterUtils.nameFileFilter(String name)`。
### 行迭代器
- `LineIterator`
  - 用于逐行迭代文件内容。
### 字符集
- `Charsets`
  - 提供对常用字符集的引用，例如 `Charsets.UTF_8`。
### 辅助类
- `EndianUtils`
  - 用于处理不同字节序（大端和小端）的数据转换。
### 随机访问文件
- `EndianRandomAccessFile`
  - 扩展了 `RandomAccessFile`，增加了对大端和小端字节序的支持。
### 文件锁定
- `FileLock`
  - 用于锁定文件的一部分或全部。
以下是一个使用 `FileUtils` 的简单示例，用于复制文件：
```java
import org.apache.commons.io.FileUtils;
public class CommonsIoExample {
    public static void main(String[] args) {
        try {
            FileUtils.copyFile(new File("source.txt"), new File("destination.txt"));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## Hutool
Hutool是一个小而全的Java工具类库，它简化了Java开发中的很多复杂操作，[点击查看官网](https://hutool.cn/)。其中Hutoo-IO是Hutool库中处理IO操作的工具集。以下是Hutool-IO中一些常见的方法和工具类，按照类型分类：
### 文件和目录操作
- FileUtil
  - copy(File src, File dest, boolean isOverride): 复制文件或目录。
  - move(File src, File dest, boolean isOverride): 移动文件或目录。
  - delete(File file): 删除文件或目录。
  - mkdirs(File dir): 创建目录，包括所有必需但不存在的父目录。
  - listFileNames(String dir): 列出目录下的所有文件和文件夹名称。
### 流操作
- IoUtil
  - readBytes(InputStream in): 从输入流中读取字节。
  - readLines(InputStream in, Charset charset): 从输入流中按行读取字符串。
  - writeBytes(OutputStream out, byte[] data): 将字节数组写入输出流。
  - copy(InputStream in, OutputStream out): 复制输入流到输出流。
  - closeQuietly(Closeable closeable): 安静地关闭可关闭对象，忽略异常。
### 字符串和流转换
- StreamUtil
  - toString(InputStream in, Charset charset): 从输入流中读取字符串。
  - toStream(String content, Charset charset): 将字符串转换为输入流。
### URL和资源操作
- URLUtil
  - getURL(String url): 解析URL字符串并返回URL对象。
  - readData(String url, Charset charset): 从URL读取数据。
### ClassPath资源操作
- ClassPathResource
  - getResource(String resource): 获取ClassPath下的资源。
  - readString(String resource, Charset charset): 读取ClassPath下的资源为字符串。
### 文件监听
- WatchMonitor
  - 用于监听文件或目录的变化。
### 文件编码
- CharsetUtil
  - 提供了常用的字符集常量，例如 CharsetUtil.UTF_8。
以下是一个使用Hutool-IO的简单示例，用于复制文件：
```java
import cn.hutool.core.io.FileUtil;
import cn.hutool.core.io.IoUtil;
public class HutoolIoExample {
    public static void main(String[] args) {
        try {
            FileUtil.copy(new File("source.txt"), new File("destination.txt"), true);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```


