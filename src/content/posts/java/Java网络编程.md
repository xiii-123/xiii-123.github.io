---
title: Java网络编程
published: 2025-03-15
description: 'Java的网络编程相比Python，Go的要麻烦许多'
image: ''
tags: [java, 网络编程]
category: 'java'
draft: false 
---
# UDP
Java中的UDP通信是一种无连接的网络通信协议，它允许应用程序发送和接收数据报文，但不保证数据报文的可靠传输。UDP通常用于那些对实时性要求高，但可以容忍一定丢包率的场景，如视频流、在线游戏等。
## 使用到的类：
1. **DatagramSocket**：用于发送和接收UDP数据报文。可以指定端口号和绑定到本地的地址。
2. **DatagramPacket**：代表UDP数据报文。每个数据报文包含发送或接收的数据以及远程主机的IP地址和端口号。
## 注意事项：
1. **数据完整性**：UDP不保证数据报文的顺序和完整性，需要应用层来实现。
2. **数据大小限制**：UDP数据报文的大小受限于网络MTU（最大传输单元），通常为1500字节。
3. **异常处理**：网络通信中可能会出现多种异常，如`SocketException`、`IOException`等，需要妥善处理。
4. **端口占用**：确保使用的端口号没有被其他应用程序占用。
5. **安全性**：UDP通信不提供加密和认证机制，敏感数据需要在应用层进行加密处理。
## 最佳实践示例：
以下是一个简单的UDP通信示例，包括一个UDP服务器和一个UDP客户端。服务器接收客户端发送的消息，并回送一条确认消息。
### UDP服务器：
```java
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
public class UDPServer {
    public static void main(String[] args) throws Exception {
        DatagramSocket serverSocket = new DatagramSocket(9876);
        byte[] receiveData = new byte[1024];
        
        while (true) {
            DatagramPacket receivePacket = new DatagramPacket(receiveData, receiveData.length);
            serverSocket.receive(receivePacket);
            String sentence = new String(receivePacket.getData(), 0, receivePacket.getLength());
            System.out.println("接收到: " + sentence);
            
            InetAddress IPAddress = receivePacket.getAddress();
            int port = receivePacket.getPort();
            String capitalizedSentence = sentence.toUpperCase();
            byte[] sendData = capitalizedSentence.getBytes();
            
            DatagramPacket sendPacket = new DatagramPacket(sendData, sendData.length, IPAddress, port);
            serverSocket.send(sendPacket);
        }
    }
}
```
### UDP客户端：
```java
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.util.Scanner;
public class UDPClient {
    public static void main(String[] args) throws Exception {
        DatagramSocket clientSocket = new DatagramSocket();
        InetAddress IPAddress = InetAddress.getByName("localhost");
        Scanner scanner = new Scanner(System.in);
        
        while (true) {
            System.out.print("输入一句话: ");
            String sentence = scanner.nextLine();
            byte[] sendData = sentence.getBytes();
            
            DatagramPacket sendPacket = new DatagramPacket(sendData, sendData.length, IPAddress, 9876);
            clientSocket.send(sendPacket);
            
            byte[] receiveData = new byte[1024];
            DatagramPacket receivePacket = new DatagramPacket(receiveData, receiveData.length);
            clientSocket.receive(receivePacket);
            
            String modifiedSentence = new String(receivePacket.getData(), 0, receivePacket.getLength());
            System.out.println("从服务器接收到: " + modifiedSentence);
        }
    }
}
```

# TCP
Java中的TCP通信是一种面向连接的、可靠的、基于字节流的传输层通信协议。它确保了数据传输的顺序性和完整性，适用于需要高可靠性的应用场景。
## 使用到的类：
1. `Socket`：用于客户端创建连接到指定服务器和端口号的对象。
2. `ServerSocket`：用于服务器端监听客户端的连接请求，并创建`Socket`对象与客户端进行通信。
3. `InputStream` 和 `OutputStream`：用于从`Socket`读取或写入数据。
## 注意事项：
1. 异常处理：网络通信可能会遇到多种异常，如`IOException`、`SocketTimeoutException`等，需要妥善处理。
2. 资源释放：使用完`Socket`和`ServerSocket`后，需要正确关闭它们以释放资源。
3. 多线程处理：服务器端通常会为每个客户端连接创建一个线程，以处理并发请求。
4. 性能考虑：大量并发连接时，应考虑使用线程池来优化性能。
5. 数据完整性：确保发送和接收的数据格式一致，以正确解析数据。
## 最佳实践示例：
以下是一个简单的TCP通信示例，包括一个TCP服务器和一个TCP客户端。服务器监听一个端口，等待客户端的连接请求，然后接收客户端发送的消息，并将消息回送给客户端。
### TCP服务器：
```java
import java.io.*;
import java.net.*;
public class TCPServer {
    public static void main(String[] args) throws IOException {
        int port = 1234;
        ServerSocket serverSocket = new ServerSocket(port);
        System.out.println("服务器启动，等待连接...");
        try {
            Socket clientSocket = serverSocket.accept();
            System.out.println("客户端连接成功！");
            try {
                PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true);
                BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
                String inputLine;
                while ((inputLine = in.readLine()) != null) {
                    System.out.println("接收到客户端: " + inputLine);
                    out.println("服务器回声: " + inputLine);
                }
            } finally {
                clientSocket.close();
            }
        } finally {
            serverSocket.close();
        }
    }
}
```
### TCP客户端：
```java
import java.io.*;
import java.net.*;
public class TCPClient {
    public static void main(String[] args) throws IOException {
        String hostName = "localhost";
        int portNumber = 1234;
        try (
            Socket echoSocket = new Socket(hostName, portNumber);
            PrintWriter out = new PrintWriter(echoSocket.getOutputStream(), true);
            BufferedReader in = new BufferedReader(new InputStreamReader(echoSocket.getInputStream()));
            BufferedReader stdIn = new BufferedReader(new InputStreamReader(System.in))
        ) {
            String userInput;
            while ((userInput = stdIn.readLine()) != null) {
                out.println(userInput);
                System.out.println("服务器回声: " + in.readLine());
            }
        } catch (UnknownHostException e) {
            System.err.println("无法识别的主机: " + hostName);
            System.exit(1);
        } catch (IOException e) {
            System.err.println("无法连接到主机: " + hostName);
            System.exit(1);
        }
    }
}
```