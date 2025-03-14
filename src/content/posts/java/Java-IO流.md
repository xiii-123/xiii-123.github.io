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