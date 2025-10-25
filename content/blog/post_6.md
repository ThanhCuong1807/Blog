---
title: "Chương 6: Kĩ thuật đa tiến trình và tuần tự hóa đối tượng trong ứng dụng mạng"
date: 2025-10-24
tags: ["mollis urna", "notes", "laoreet"]
categories: ["diam aliqua", "fringilla"]
description: "dictum tellus sapien vitae integer justo amet mauris cras bolestie sollicitudin dignissim"
draft: false
---

## I. Giới thiệu: Nâng cấp Server
### 1. Mục tiêu bài học
* Tạo new Thread() cho mỗi Client là rất lãng phí và nguy hiểm.
* Server chỉ có thể gửi/nhận dữ liệu String (văn bản) thô.
* Học cách dùng Thread Pool (ExecutorService) để quản lý luồng hiệu quả, giới hạn số lượng luồng chạy song song (một kĩ thuật quản lý "đa tiến trình" hiệu quả).
* Học cách Tuần tự hóa (Serialize) một đối tượng Java (biến nó thành byte) để gửi qua mạng.
* Nâng cấp Server/Client để có thể gửi và nhận các đối tượng Java (ví dụ: đối tượng Message) thay vì String.

### 2. Vấn đề của Server ở Chương 5
Server đa tuyến ở chương 3 và 5 hoạt động, nhưng có 2 nhược điểm chết người:
* Cạn kiệt tài nguyên: new Thread(handler).start() sẽ tạo một luồng mới của hệ điều hành. Nếu 10,000 Client kết nối, Server sẽ cố tạo 10,000 luồng, làm sập hệ điều hành.
* Giới hạn dữ liệu: Chúng ta dùng PrintWriter và BufferedReader để gửi String. Nếu muốn gửi một đối tượng phức tạp (như thông tin sinh viên, một tin nhắn có kèm tên người gửi, thời gian...), chúng ta phải tự "mã hóa" String đó (ví dụ: username;timestamp;content) rồi "giải mã" ở phía bên kia. Rất phiền phức.
## II. Kĩ thuật đa tiến trình/luồng hiệu quả: Thread Pool
Để giải quyết vấn đề 1, chúng ta không "thuê" một "người phục vụ" (Thread) mới cho mỗi "vị khách" (Client). Thay vào đó, chúng ta thuê một số lượng nhân viên cố định (ví dụ: 10 người) và tạo một hàng đợi (Queue) cho khách.
* 10 "người phục vụ" (luồng) này sẽ liên tục lấy "vị khách" (Client) tiếp theo trong hàng đợi để phục vụ.
* Dù có 10,000 khách kết nối, chúng ta vẫn chỉ dùng 10 luồng. 9,990 khách kia sẽ xếp hàng chờ (rất nhanh) để được phục vụ.
Đây gọi là Thread Pool (Bể luồng). Trong Java, chúng ta dùng ExecutorService.
{{< figure src="/Blog/images/bai6.webp" alt="Sơ đồ minh họa Thread Pool" caption="Hình 1: So sánh Thread-per-Client (trái) và Thread Pool (phải). Thread Pool hiệu quả hơn nhiều." >}}
**Nâng cấp Server.java dùng Thread Pool**  
Chúng ta chỉ cần sửa lại Server.java (lớp "quản lý"). Lớp ClientHandler.java không cần thay đổi logic.  

'''java

    import java.net.ServerSocket;
    import java.net.Socket;
    import java.util.concurrent.ExecutorService;
    import java.util.concurrent.Executors;
    public class Server {
        public static final int PORT = 9999;
        public static final int NUM_THREADS = 10; // Số lượng "nhân viên" cố định

        public static void main(String[] args) {
            
            // 1. Tạo một Thread Pool với 10 luồng
            ExecutorService executor = Executors.newFixedThreadPool(NUM_THREADS);

            try {
                ServerSocket serverSocket = new ServerSocket(PORT);
                System.out.println("Server đang chạy ở cổng " + PORT + "...");
                System.out.println("Sử dụng Thread Pool với " + NUM_THREADS + " luồng.");

                while (true) {
                    // 2. Chấp nhận kết nối mới
                    Socket clientSocket = serverSocket.accept();
                    System.out.println("Client mới kết nối: " + clientSocket.getInetAddress());

                    // 3. Thay vì tạo Thread mới...
                    // new Thread(new ClientHandler(clientSocket)).start(); // (Cách cũ)

                    // ...Chúng ta "giao" công việc cho Thread Pool
                    executor.submit(new ClientHandler(clientSocket));
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
## III. Tuần tự hóa Đối tượng (Object Serialization)
Để giải quyết vấn đề 2 (gửi đối tượng), chúng ta dùng kĩ thuật "Tuần tự hóa".
* Serialization (Tuần tự hóa): Biến một đối tượng Java (Object) trong bộ nhớ RAM thành một dòng byte (stream of bytes).
* Deserialization (Giải tuần tự hóa): Biến dòng byte đó ngược trở lại thành một đối tượng Java.
{{< figure src="/Blog/images/bai6_2.png" alt="Sơ đồ Tuần tự hóa đối tượng" caption="Hình 2: Quá trình một Object được 'tuần tự hóa' thành byte để gửi đi và 'giải tuần tự hóa' lại." >}}

Để gửi một đối tượng, chúng ta cần:
* Lớp của đối tượng đó phải implements java.io.Serializable.
* Dùng ObjectOutputStream (thay cho PrintWriter) để gửi.
* Dùng ObjectInputStream (thay cho BufferedReader) để nhận.  
**1. Tạo Lớp đối tượng để gửi (Message.java)**  
Chúng ta sẽ tạo một lớp Message đơn giản.

'''java

    import java.io.Serializable;

    // 1. Phải "implements Serializable"
    public class Message implements Serializable {

        // 2. Cần có serialVersionUID để đảm bảo 2 bên dùng chung 1 phiên bản
        private static final long serialVersionUID = 1L; 

        private String sender;
        private String content;

        public Message(String sender, String content) {
            this.sender = sender;
            this.content = content;
        }

        // (Thêm các hàm getter, setter...)
        public String getSender() { return sender; }
        public String getContent() { return content; }

        @Override
        public String toString() {
            return sender + " nói: " + content;
        }
**2. Nâng cấp ClientHandler.java (dùng Object Streams)**  
Bây giờ "người phục vụ" sẽ không đọc/ghi String nữa, mà đọc/ghi Message.
'''java
    import java.io.*;
    import java.net.Socket;

    public class ClientHandler implements Runnable {
        private Socket clientSocket;
        private ObjectInputStream in;  // <-- THAY ĐỔI
        private ObjectOutputStream out; // <-- THAY ĐỔI

        public ClientHandler(Socket socket) {
            this.clientSocket = socket;
        }

        @Override
        public void run() {
            try {
                // !!! LƯU Ý QUAN TRỌNG:
                // Phải tạo ObjectOutputStream TRƯỚC ObjectInputStream
                // nếu không sẽ bị "treo" (deadlock)
                out = new ObjectOutputStream(clientSocket.getOutputStream());
                in = new ObjectInputStream(clientSocket.getInputStream());

                // --- Logic chính: Đọc Object ---
                Message messageFromClient;
                
                // Đọc một đối tượng Message
                while ((messageFromClient = (Message) in.readObject()) != null) { 
                    System.out.println("Client " + clientSocket.getPort() + " - " + messageFromClient);

                    // Phản hồi lại Client (gửi 1 object)
                    Message serverResponse = new Message("Server", "Đã nhận được: " + messageFromClient.getContent());
                    out.writeObject(serverResponse);
                }

            } catch (EOFException e) {
                System.out.println("Client " + clientSocket.getPort() + " đã ngắt kết nối (EOF).");
            } catch (Exception e) {
                System.out.println("Lỗi ClientHandler: " + e.getMessage());
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
**3. Nâng cấp Client.java (dùng Object Streams)**  
Client giờ cũng sẽ gửi đi đối tượng Message.
'''java
    import java.io.*;
    import java.net.Socket;
    import java.util.Scanner;

    public class Client {
        public static final String SERVER_HOST = "localhost";
        public static final int SERVER_PORT = 9999;

        public static void main(String[] args) {
            try {
                Socket socket = new Socket(SERVER_HOST, SERVER_PORT);
                System.out.println("Đã kết nối tới Server...");

                // Phải tạo ObjectOutputStream TRƯỚC
                ObjectOutputStream out = new ObjectOutputStream(socket.getOutputStream());
                ObjectInputStream in = new ObjectInputStream(socket.getInputStream());
                
                Scanner scanner = new Scanner(System.in);
                String userInput;

                while (true) {
                    System.out.print("Nhập tin nhắn (gõ 'exit' để thoát): ");
                    userInput = scanner.nextLine();

                    Message messageToSend;
                    if ("exit".equalsIgnoreCase(userInput)) {
                        messageToSend = new Message("Client", "exit");
                        out.writeObject(messageToSend); // Gửi đối tượng
                        break;
                    }
                    
                    messageToSend = new Message("Client", userInput);
                    out.writeObject(messageToSend); // Gửi đối tượng

                    // Chờ và đọc phản hồi (là một Object)
                    Message serverResponse = (Message) in.readObject();
                    System.out.println("Server phản hồi: " + serverResponse.getContent());
                }

                scanner.close();
                in.close();
                out.close();
                socket.close();

            } catch (Exception e) {
                e.printStackTrace();
            }
        }
