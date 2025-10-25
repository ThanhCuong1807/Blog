---
title: "Chương 5: Lập trình sockets cho giao thức TCP"
date: 2025-10-24
tags: ["mollis urna", "notes", "laoreet"]
categories: ["diam aliqua", "fringilla"]
description: "dictum tellus sapien vitae integer justo amet mauris cras bolestie sollicitudin dignissim"
draft: false
---

## I. Giới thiệu: Ghép nối các mảnh ghép
### 1. Mục tiêu bài học
* Hiểu rõ vai trò và sự khác biệt của hai lớp Socket TCP chính: ServerSocket (phía Server) và Socket (phía Client).
* Vận dụng kiến thức Chương 3 (Đa tuyến) để xây dựng một Server TCP có khả năng chấp nhận nhiều Client cùng lúc.
* Vận dụng kiến thức Chương 2 (Streams) để Client và Server có thể gửi và nhận dữ liệu (văn bản) cho nhau.
* Vận dụng kiến thức Chương 4 (InetAddress) để Client biết cách tìm và kết nối đến Server.
### 2. Tổng quan về Sockets TCP
* Lập trình Socket TCP là mô hình Client-Server (Chương 1) sử dụng giao thức TCP.
* Server (Bên lắng nghe): Mở một "cổng" (Port) trên máy mình và chờ. Nó sử dụng lớp ServerSocket.
* Client (Bên kết nối): Chủ động tìm đến "địa chỉ" (IP) và "cổng" (Port) của Server để xin kết nối. Nó sử dụng lớp Socket.
* Kết nối: Khi Server "chấp nhận" (accept()) Client, một kênh giao tiếp 2 chiều, tin cậy (TCP) được thiết lập. Cả hai bên lúc này đều dùng đối tượng Socket để gửi và nhận dữ liệu qua I/O Streams (Bài 2).

{{< figure src="/Blog/images/bai5.gif" alt="Sơ đồ minh họa bắt tay 3 bước TCP" caption="Hình 1: Quy trình Server 'mở cổng' (bind/listen) và Client 'kết nối' (connect)." >}}
## II. Lập trình Server (Phía lắng nghe)
Phía Server cần làm 2 việc:
* Luôn lắng nghe kết nối mới (dùng ServerSocket).
* Với mỗi kết nối mới, giao nó cho một luồng riêng (dùng Thread - Chương 3) để xử lý.
**1. Lớp ServerSocket**
Đây là lớp chỉ dành cho Server. Nhiệm vụ duy nhất của nó là "ngồi" tại một cổng và chờ "tiếng gõ cửa" (yêu cầu kết nối) từ Client.
* new ServerSocket(port): "Mở" một cổng trên máy chủ.
* serverSocket.accept(): Một hàm blocking (khóa). Nó sẽ "treo" luồng chính lại và chờ cho đến khi có một Client kết nối đến. Khi có Client, nó trả về một đối tượng Socket đại diện cho kết nối đó.
**2. Code ví dụ: Server (Đa tuyến)**
**Lớp Server.java (Người quản lý)**
'''java
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

                    // 3. Giao client này cho 1 luồng xử lý riêng (ClientHandler)
                    ClientHandler clientHandler = new ClientHandler(clientSocket);
                    Thread thread = new Thread(clientHandler);
                    thread.start(); 
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    
**Lớp ClientHandler.java (Người phục vụ)**
'''java
    import java.io.*;
    import java.net.Socket;

    public class ClientHandler implements Runnable {
        private Socket clientSocket;
        private BufferedReader in; // Đọc từ Client
        private PrintWriter out;   // Ghi cho Client

        public ClientHandler(Socket socket) {
            this.clientSocket = socket;
        }

        @Override
        public void run() {
            try {
                // Thiết lập luồng đọc (Kịch bản 1 - Bài 2)
                in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream(), "UTF-8"));
                
                // Thiết lập luồng ghi (Kịch bản 2 - Bài 2)
                out = new PrintWriter(new OutputStreamWriter(clientSocket.getOutputStream(), "UTF-8"), true);

                String messageFromClient;
                while ((messageFromClient = in.readLine()) != null) {
                    System.out.println("Client " + clientSocket.getPort() + " nói: " + messageFromClient);
                    out.println("Server phản hồi: " + messageFromClient); // Gửi lại (Echo)
                }
            } catch (IOException e) {
                System.out.println("Client " + clientSocket.getPort() + " đã ngắt kết nối.");
            } finally {
                try {
                    if (in != null) in.close();
                    if (out != null) out.close();
                    if (clientSocket != null) clientSocket.close();
                } catch (IOException ex) {
                    ex.printStackTrace();
                }
            }
        }
    
## III. Lập trình Client (Phía kết nối)
Phía Client đơn giản hơn:
* Biết địa chỉ và cổng của Server (dùng kiến thức Chương 4).
* Chủ động kết nối đến Server (dùng Socket).
* Sau khi kết nối thành công, dùng I/O Streams (Chương 2) để gửi và nhận dữ liệu.
{{< figure src="/Blog/images/bai5-1.webp" alt="Sơ đồ kiến trúc Client-Server đa tuyến" caption="Hình 2: Luồng hoạt động của Server đa tuyến (Bài 3) và Client (Bài 5)." >}}
### 1. Lớp Socket
Đây là lớp dành cho Client. (Server cũng dùng nó, nhưng là sau khi accept()).
* new Socket(String host, int port): Đây là hàm "gõ cửa". Nó sẽ tìm đến host (tên miền hoặc IP) tại port. Đây là một hàm blocking (khóa), nó sẽ "treo" chương trình Client lại cho đến khi Server chấp nhận.
* socket.getInputStream(): Lấy luồng để đọc dữ liệu Server gửi.
* socket.getOutputStream(): Lấy luồng để ghi dữ liệu gửi cho Server.
'''java
    import java.io.*;
    import java.net.Socket;
    import java.util.Scanner;

    public class Client {
        public static final String SERVER_HOST = "localhost"; // Địa chỉ Server
        public static final int SERVER_PORT = 9999;          // Cổng Server

        public static void main(String[] args) {
            try {
                // 1. Kết nối đến Server
                // (Sử dụng kiến thức Bài 4, "localhost" sẽ được DNS tra cứu)
                Socket socket = new Socket(SERVER_HOST, SERVER_PORT);
                System.out.println("Đã kết nối tới Server: " + socket.getInetAddress());

                // 2. Thiết lập luồng đọc từ Server (như Bài 2)
                BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream(), "UTF-8"));
                
                // 3. Thiết lập luồng ghi tới Server (như Bài 2)
                PrintWriter out = new PrintWriter(new OutputStreamWriter(socket.getOutputStream(), "UTF-8"), true);

                // 4. Thiết lập luồng đọc từ bàn phím người dùng
                Scanner scanner = new Scanner(System.in);

                String userInput;
                String serverResponse;

                // Vòng lặp chính của Client
                while (true) {
                    // Đọc từ bàn phím
                    System.out.print("Nhập tin nhắn (gõ 'exit' để thoát): ");
                    userInput = scanner.nextLine();

                    // Gửi tin nhắn lên Server
                    out.println(userInput);

                    // Nếu gõ 'exit' thì thoát
                    if ("exit".equalsIgnoreCase(userInput)) {
                        break;
                    }

                    // Chờ và đọc phản hồi từ Server
                    serverResponse = in.readLine();
                    System.out.println("Server phản hồi: " + serverResponse);
                }

                // 5. Đóng kết nối
                scanner.close();
                in.close();
                out.close();
                socket.close();

            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    