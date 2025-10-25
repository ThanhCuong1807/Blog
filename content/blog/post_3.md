---
title: "Chương 3: Lập trình đa tuyến"
date: 2018-01-06
tags: ["mollis urna", "notes", "laoreet"]
categories: ["diam aliqua", "fringilla"]
description: "dictum tellus sapien vitae integer justo amet mauris cras bolestie sollicitudin dignissim"
draft: false
---

## I. Giới thiệu: Vấn đề của Server đơn tuyến

### 1. Mục tiêu bài học
* Hiểu rõ khái niệm **Tuyến** (Thread) và **Đa tuyến** (Multithreading).
* Giải thích được tại sao lập trình mạng **bắt buộc** phải sử dụng đa tuyến.
* Nắm được hai cách tạo Thread trong Java: Kế thừa lớp `Thread` và triển khai `Runnable` (Interface).
* Áp dụng đa tuyến để xây dựng một **Server** hoàn chỉnh có khả năng xử lý nhiều Client cùng lúc mà không bị "khóa".

### 2. Vấn đề "Blocking"
Ở **Bài 2**, chúng ta đã thấy Kịch bản 1 (Đọc) sử dụng hàm `bufferedReader.readLine()`. Vấn đề là: hàm này là một hàm **"blocking" (khóa)**.

Nó sẽ "treo" toàn bộ chương trình của bạn lại và **chờ** cho đến khi Client gửi một dòng dữ liệu.

Hãy tưởng tượng một Server viết theo kiểu đơn giản:
```java
// Mã giả của Server đơn tuyến (SAI)

// 1. Chờ Client 1 kết nối
Socket client1 = serverSocket.accept(); // <-- Bị khóa chờ Client 1

// 2. Chờ Client 1 nói chuyện
String msg1 = bufferedReader1.readLine(); // <-- Bị khóa chờ Client 1 gửi tin

// 3. Chờ Client 2 kết nối
Socket client2 = serverSocket.accept(); // <-- KHÔNG BAO GIỜ CHẠY ĐƯỢC
```
## II. Thread và Multithreading là gì?
### 1. Khái niệm Thread
Hãy tưởng tượng Process (tiến trình) là một nhà hàng. Khi bạn chạy ứng dụng Server.java, bạn đã mở một "nhà hàng".
* Một Thread (tuyến) giống như một người phục vụ trong nhà hàng đó.
* Server đơn tuyến (Single-thread): Nhà hàng của bạn chỉ có một người phục vụ. Người này phải chạy ra cửa đón 8   khách (accept), sau đó vào bếp chờ khách gọi món (readLine), rồi mới đi giao món (write). Khi đang làm vậy, họ không thể đón thêm khách mới.
* Server đa tuyến (Multi-thread): Nhà hàng của bạn có nhiều người phục vụ.
* Có một người quản lý (main thread) chỉ đứng ở cửa để đón khách (accept()).
* Mỗi khi có khách mới, quản lý sẽ "quẳng" vị khách đó cho một người phục vụ mới (new Thread(handler)) để chăm sóc riêng.
* Người quản lý ngay lập tức quay lại cửa để đón khách tiếp theo.
* Đa tuyến (Multithreading) là khả năng một Process (ứng dụng) có thể thực thi nhiều Thread (tác vụ) song song cùng một lúc.
{{< figure src="/Blog/images/bai3.jpg" alt="Sơ đồ minh họa server đơn tuyến vs đa tuyến" caption="Hình 1: Server đơn tuyến (trái) bị khóa và Server đa tuyến (phải) xử lý song song." >}}
### 2. Cách tạo Thread trong Java
Có 2 cách chính để tạo một "người phục vụ" mới:  
**Cách 1: Kế thừa lớp Thread**
```java
class ClientHandler extends Thread {
    public void run() {
        // Đặt code xử lý client vào đây
        System.out.println("Một client đang được xử lý trên luồng riêng!");
    }
}
// Cách sử dụng:
ClientHandler handler = new ClientHandler();
handler.start(); // Yêu cầu Thread bắt đầu chạy (sẽ gọi hàm run())
```
**Cách 2: Triển khai Interface Runnable (Khuyến khích)**  
Bạn tạo một lớp triển khai (implement) Runnable. Đây là cách làm tốt hơn vì Java không hỗ trợ đa kế thừa (lớp của bạn vẫn có thể kế thừa một lớp khác nếu cần).
```java
class ClientHandler implements Runnable {
    public void run() {
        // Đặt code xử lý client vào đây
        System.out.println("Một client đang được xử lý trên luồng riêng!");
    }
}
// Cách sử dụng:
ClientHandler handler = new ClientHandler();
Thread thread = new Thread(handler); // "Giao" công việc cho một Thread
thread.start(); // Yêu cầu Thread bắt đầu
```
## III. Áp dụng: Xây dựng Server đa tuyến
Bây giờ, chúng ta sẽ viết lại code Server hoàn chỉnh, áp dụng Cách 2 (Runnable). Chúng ta sẽ chia code làm 2 lớp:  
**1. Lớp Server chính (Server.java)**  
Lớp này đóng vai trò "người quản lý". Nhiệm vụ duy nhất là đứng ở cửa (accept()) và giao client mới cho một ClientHandler.
```java
import java.net.ServerSocket;
import java.net.Socket;

public class Server {
    public static final int PORT = 9999;

    public static void main(String[] args) {
        try {
            // 1. Mở cổng server
            ServerSocket serverSocket = new ServerSocket(PORT);
            System.out.println("Server đang chạy ở cổng " + PORT + "...");

            // 2. Luôn luôn chờ kết nối mới (vòng lặp vô tận)
            while (true) {
                // Hàm accept() sẽ "khóa" luồng main cho đến khi có client
                Socket clientSocket = serverSocket.accept();
                System.out.println("Client mới kết nối: " + clientSocket.getInetAddress());

                // 3. Tạo một "người phục vụ" (ClientHandler)
                ClientHandler clientHandler = new ClientHandler(clientSocket);

                // 4. "Giao" client cho người phục vụ đó và yêu cầu bắt đầu
                Thread thread = new Thread(clientHandler);
                thread.start(); // Server ngay lập tức quay lại vòng lặp để chờ client tiếp theo
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
**2. Lớp Xử lý Client (ClientHandler.java)**  
Lớp này là "người phục vụ", chứa logic của Kịch bản 1 (Đọc) và Kịch bản 2 (Ghi) từ Bài 2. Mỗi client sẽ có 1 đối tượng ClientHandler chạy trên 1 Thread riêng.
```java
import java.io.*;
import java.net.Socket;

// Triển khai Runnable để có thể chạy trên 1 Thread
public class ClientHandler implements Runnable {

    private Socket clientSocket;
    private BufferedReader in; // Kịch bản 1: Đọc
    private PrintWriter out;   // Kịch bản 2: Ghi

    public ClientHandler(Socket socket) {
        this.clientSocket = socket;
    }

    @Override
    public void run() {
        // Hàm run() này sẽ chạy trên một luồng riêng biệt
        try {
            // === THIẾT LẬP KỊCH BẢN 1 (ĐỌC) ===
            InputStream inputStream = clientSocket.getInputStream();
            Reader reader = new InputStreamReader(inputStream, "UTF-8");
            in = new BufferedReader(reader);

            // === THIẾT LẬP KỊCH BẢN 2 (GHI) ===
            OutputStream outputStream = clientSocket.getOutputStream();
            Writer writer = new OutputStreamWriter(outputStream, "UTF-8");
            out = new PrintWriter(writer, true); // true = autoFlush

            // --- Logic chính: Hỏi-Đáp (Echo Server) ---
            String messageFromClient;
            
            // Kịch bản 1: Đọc (hàm readLine() khóa luồng NÀY)
            // Luồng main của server không bị ảnh hưởng
            while ((messageFromClient = in.readLine()) != null) {
                System.out.println("Client " + clientSocket.getPort() + " nói: " + messageFromClient);

                // Kịch bản 2: Ghi (Phản hồi lại client)
                out.println("Server phản hồi: " + messageFromClient);
            }

        } catch (IOException e) {
            System.out.println("Client " + clientSocket.getPort() + " đã ngắt kết nối.");
        } finally {
            // Dọn dẹp tài nguyên khi client ngắt kết nối
            try {
                if (in != null) in.close();
                if (out != null) out.close();
                if (clientSocket != null) clientSocket.close();
            } catch (IOException ex) {
                ex.printStackTrace();
            }
        }
    }
}
```