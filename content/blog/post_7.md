---
title: "Chương 7: Lập trình mạng cho giao thức UDP"
date: 2025-10-24
tags: ["mollis urna", "notes", "laoreet"]
categories: ["diam aliqua", "fringilla"]
description: "dictum tellus sapien vitae integer justo amet mauris cras bolestie sollicitudin dignissim"
draft: false
---

## I. Giới thiệu: Lập trình phi kết nối (UDP)
### 1. Mục tiêu bài học
* Hiểu rõ sự khác biệt triệt để giữa lập trình TCP (Chương 5) và UDP.
* Nắm vững hai lớp Socket UDP chính: DatagramSocket và DatagramPacket.
* Biết cách "gói" và "mở" dữ liệu (mảng byte[]) vào các gói tin.
* Xây dựng được một Server UDP (Echo Server) có khả năng nhận và phản hồi gói tin.
* Xây dựng được một Client UDP có khả năng gửi và nhận gói tin.
### 2. Ôn lại về UDP (Từ Chương 1)
Khác với TCP (giống cuộc gọi điện thoại), UDP giống như gửi bưu thiếp:
* Phi kết nối (Connectionless): Không cần "bắt tay" 3 bước. Client cứ thế "ném" gói tin đi, Server cứ thế nhận.
* Không tin cậy (Unreliable): Không có gì đảm bảo gói tin sẽ đến nơi, đến đúng thứ tự, hay không bị lặp.
* Nhanh (Fast): Vì không tốn chi phí quản lý kết nối, UDP rất nhanh.
Dựa trên Datagram (Gói): Dữ liệu được gửi/nhận dưới dạng các "gói tin" (DatagramPacket) rời rạc.
{{< figure src="/Blog/images/bai7.webp" alt="Sơ đồ minh họa UDP gửi bưu thiếp" caption="Hình 1: Client 'gửi bưu thiếp' (packet) mà không cần thiết lập kết nối trước." >}}
## II. Các lớp Socket UDP
Chúng ta không dùng Socket hay ServerSocket. Thay vào đó, chúng ta dùng:  
**1. DatagramSocket (Ổ cắm Datagram):**
* Tương tự "Bưu điện". Đây là nơi bạn đứng để gửi hoặc nhận "bưu thiếp".
* Server: new DatagramSocket(PORT) -> Mở một "bưu điện" tại cổng PORT để chờ nhận thư.
* Client: new DatagramSocket() -> Mở một "bưu điện" tại một cổng ngẫu nhiên (do HĐH cấp) để gửi thư đi.

**2. DatagramPacket (Gói Datagram):**

* Tương tự "Bưu thiếp" (hoặc "phong bì").
* Đây là đối tượng chứa dữ liệu (byte[]) và thông tin địa chỉ (IP + Port) của người gửi/người nhận.
* Khi gửi: Bạn tạo 1 DatagramPacket chứa dữ liệu VÀ địa chỉ người nhận (new DatagramPacket(data, data.length, address, port)).
* Khi nhận: Bạn tạo 1 DatagramPacket rỗng (chỉ có 1 mảng byte rỗng làm bộ đệm) để hứng dữ liệu (new DatagramPacket(buffer, buffer.length)).
## III. Lập trình Server (Phía nhận)
Server UDP thường là một vòng lặp while(true) đơn giản:
* Tạo một bộ đệm (buffer) và 1 packet rỗng.
* Chờ nhận (socket.receive()). Đây là hàm blocking.
* Khi nhận được, "mở" packet ra xem: lấy dữ liệu VÀ lấy địa chỉ người gửi.
* Tạo packet phản hồi, nhét dữ liệu vào, và địa chỉ của người gửi (lấy từ bước 3).
* Gửi phản hồi (socket.send()).
* Lặp lại.  
**Lưu ý:** Server UDP không cần đa tuyến (Thread Pool) cho trường hợp đơn giản. Vì receive() nhận gói tin từ bất kỳ ai, không giống ServerSocket bị "khóa" với 1 client trong lúc accept().  
Code ví dụ: **UDPEchoServer.java**  

'''java

    import java.net.DatagramPacket;
    import java.net.DatagramSocket;
    import java.net.InetAddress;
    public class UDPEchoServer {
        public static final int PORT = 9998;
        public static final int BUFFER_SIZE = 1024; // Bộ đệm 1KB

        public static void main(String[] args) {
            try {
                // 1. Mở "bưu điện" tại cổng 9998
                DatagramSocket socket = new DatagramSocket(PORT);
                System.out.println("UDP Server đang chạy ở cổng " + PORT + "...");

                // 2. Tạo một bộ đệm (mảng byte) để nhận dữ liệu
                byte[] receiveBuffer = new byte[BUFFER_SIZE];

                while (true) {
                    // 3. Tạo 1 "phong bì rỗng" để nhận dữ liệu
                    DatagramPacket receivePacket = new DatagramPacket(receiveBuffer, receiveBuffer.length);
                    
                    // 4. Chờ nhận "bưu thiếp" (blocking)
                    socket.receive(receivePacket);

                    // 5. "Mở bưu thiếp"
                    // Lấy dữ liệu từ bộ đệm
                    String clientMessage = new String(receivePacket.getData(), 0, receivePacket.getLength());
                    
                    // Lấy địa chỉ của người gửi
                    InetAddress clientAddress = receivePacket.getAddress();
                    int clientPort = receivePacket.getPort();
                    System.out.println("Nhận từ [" + clientAddress.getHostAddress() + ":" + clientPort + "]: " + clientMessage);
                    
                    // 6. Chuẩn bị "bưu thiếp" phản hồi
                    String serverResponse = "Server phản hồi: " + clientMessage;
                    byte[] sendBuffer = serverResponse.getBytes("UTF-8");

                    DatagramPacket sendPacket = new DatagramPacket(
                            sendBuffer, 
                            sendBuffer.length, 
                            clientAddress, // Gửi về đúng địa chỉ người gửi
                            clientPort     // Gửi về đúng cổng của người gửi
                    );

                    // 7. Gửi phản hồi
                    socket.send(sendPacket);
                }

            } catch (Exception e) {
                e.printStackTrace();
            }
            // Server này không bao giờ đóng (trừ khi có lỗi)
        }
    }
## IV. Lập trình Client (Phía gửi)
Client UDP cũng đơn giản:
* Mở "bưu điện" ở cổng ngẫu nhiên (new DatagramSocket()).
* Lấy địa chỉ Server (dùng InetAddress).
* Tạo dữ liệu, nhét vào "bưu thiếp" (DatagramPacket) có ghi địa chỉ Server.
* Gửi đi (socket.send()).
* Tạo "bưu thiếp rỗng" chờ thư phản hồi (socket.receive()).
* In ra.  
Code ví dụ: **UDPEchoClient.java**  

'''java

    import java.net.DatagramPacket;
    import java.net.DatagramSocket;
    import java.net.InetAddress;
    import java.util.Scanner;

    public class UDPEchoClient {
        public static final String SERVER_HOST = "localhost";
        public static final int SERVER_PORT = 9998; // Phải đúng cổng Server
        public static final int BUFFER_SIZE = 1024;

        public static void main(String[] args) {
            try {
                // 1. Mở "bưu điện" ở cổng ngẫu nhiên
                DatagramSocket socket = new DatagramSocket();
                
                // 2. Lấy địa chỉ IP của Server (Bài 4)
                InetAddress serverAddress = InetAddress.getByName(SERVER_HOST);
                
                Scanner scanner = new Scanner(System.in);
                System.out.print("Nhập tin nhắn (gõ 'exit' để thoát): ");

                while (scanner.hasNextLine()) {
                    String userInput = scanner.nextLine();
                    if ("exit".equalsIgnoreCase(userInput)) {
                        break;
                    }

                    // 3. Tạo "bưu thiếp" để gửi
                    byte[] sendBuffer = userInput.getBytes("UTF-8");
                    DatagramPacket sendPacket = new DatagramPacket(
                            sendBuffer, 
                            sendBuffer.length, 
                            serverAddress, // Địa chỉ Server
                            SERVER_PORT    // Cổng Server
                    );
                    
                    // 4. Gửi "bưu thiếp" đi
                    socket.send(sendPacket);

                    // 5. Tạo "phong bì rỗng" chờ phản hồi
                    byte[] receiveBuffer = new byte[BUFFER_SIZE];
                    DatagramPacket receivePacket = new DatagramPacket(receiveBuffer, receiveBuffer.length);

                    // 6. Chờ nhận phản hồi (blocking)
                    socket.receive(receivePacket);
                    
                    String serverResponse = new String(receivePacket.getData(), 0, receivePacket.getLength());
                    System.out.println("Server phản hồi: " + serverResponse);
                    System.out.print("\nNhập tin nhắn (gõ 'exit' để thoát): ");
                }
                
                // 7. Đóng kết nối
                System.out.println("Client đóng kết nối.");
                scanner.close();
                socket.close();

            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }