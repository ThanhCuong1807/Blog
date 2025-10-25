---
title: "Chương 8: Lập trình multicast"
date: 2025-10-24
tags: ["mollis urna", "notes", "laoreet"]
categories: ["diam aliqua", "fringilla"]
description: "dictum tellus sapien vitae integer justo amet mauris cras bolestie sollicitudin dignissim"
draft: false
---

## I. Giới thiệu: Giao tiếp Một-Đến-Nhiều
### 1. Mục tiêu bài học
* Hiểu rõ khái niệm Multicast là gì và nó khác gì với TCP (1-đến-1) và UDP (1-đến-1).
* Nắm vững lớp MulticastSocket trong Java.
* Biết cách "tham gia" (joinGroup) và "rời khỏi" (leaveGroup) một nhóm multicast.
* Xây dựng được một Sender (Publisher) gửi tin nhắn đến một nhóm.
* Xây dựng được một Receiver (Subscriber) lắng nghe tin nhắn từ một nhóm.
### 2. Multicast là gì?
Hãy so sánh các kiểu giao tiếp:
* TCP (Chương 5): Giống cuộc gọi điện thoại (1-đến-1, tin cậy, có kết nối).
* UDP (Chương 7): Giống gửi bưu thiếp (1-đến-1, không tin cậy, phi kết nối).
* Multicast (Chương 8): Giống một đài phát thanh Radio (1-đến-nhiều, không tin cậy, phi kết nối).   

Trong mô hình multicast:
* Sender (Bên gửi): Là "đài phát thanh". Sender chỉ cần "phát" một gói tin (dùng UDP) đến một địa chỉ nhóm (Multicast Group Address) duy nhất. Sender không cần biết có ai đang nghe hay không.
* Receiver (Bên nhận): Là "máy radio". Bất kỳ ai muốn nhận tin nhắn sẽ "dò" đúng "tần số" (tức là tham gia vào địa chỉ nhóm đó).
* Hệ thống mạng (Router) sẽ tự động sao chép và gửi gói tin đến tất cả các Receiver đã tham gia.
* Địa chỉ nhóm (Multicast Address): Là các địa chỉ IP đặc biệt trong dải Class D, từ 224.0.0.0 đến 239.255.255.255.
## II. Lớp MulticastSocket
Java cung cấp lớp MulticastSocket để làm việc này. Nó là một lớp con (subclass) của DatagramSocket (Chương 7).
Nó kế thừa tất cả các phương thức send() và receive() từ DatagramSocket.  
Nó có thêm 2 phương thức quan trọng:
* joinGroup(InetAddress groupAddress): Yêu cầu card mạng bắt đầu lắng nghe các gói tin được gửi đến địa chỉ nhóm này.
* leaveGroup(InetAddress groupAddress): Ngừng lắng nghe.

Và một phương thức hữu ích khác:

* setTimeToLive(int ttl): Chỉ dùng cho Sender. Cài đặt "số bước nhảy" (hops) mà gói tin có thể đi qua. 1 (mặc định) có nghĩa là chỉ trong mạng LAN nội bộ. >1 cho phép gói tin đi qua các router.

## III. Lập trình Receiver (Subscriber - Bên nghe)
Bên nghe là bên phức tạp hơn. Nó phải chủ động tham gia vào nhóm.  
Các bước thực hiện:
* Lấy InetAddress của nhóm (ví dụ: "230.0.0.1").
* Tạo một MulticastSocket và bind nó vào một cổng (ví dụ: 8888). Đây là cổng mà nhóm sẽ giao tiếp.
* Gọi socket.joinGroup(groupAddress) để bắt đầu lắng nghe.
* Tạo một DatagramPacket rỗng (bộ đệm) và gọi socket.receive() trong một vòng lặp.
* Khi xong việc, gọi socket.leaveGroup(groupAddress).  
**Code ví dụ: MulticastReceiver.java (Bên nghe)** 

'''java

    import java.net.DatagramPacket;
    import java.net.InetAddress;
    import java.net.MulticastSocket;
    public class MulticastReceiver {
        public static final String MULTICAST_GROUP = "230.0.0.1"; // Chọn 1 địa chỉ trong dải
        public static final int PORT = 8888;
        public static final int BUFFER_SIZE = 1024;
        public static void main(String[] args) {
            MulticastSocket socket = null;
            try {
                // 1. Lấy địa chỉ IP của nhóm
                InetAddress group = InetAddress.getByName(MULTICAST_GROUP);

                // 2. Tạo MulticastSocket "nghe" trên cổng 8888
                socket = new MulticastSocket(PORT);
                
                // 3. Yêu cầu tham gia nhóm
                socket.joinGroup(group);
                System.out.println("Đã tham gia nhóm " + MULTICAST_GROUP + ", đang chờ tin nhắn...");

                byte[] buffer = new byte[BUFFER_SIZE];
                
                while (true) {
                    // 4. Tạo packet rỗng và chờ nhận (giống hệt UDP)
                    DatagramPacket receivePacket = new DatagramPacket(buffer, buffer.length);
                    socket.receive(receivePacket);

                    String message = new String(receivePacket.getData(), 0, receivePacket.getLength());
                    System.out.println("Nhận từ [" + receivePacket.getAddress() + "]: " + message);

                    // Nếu nhận được "exit", thì thoát
                    if ("exit".equalsIgnoreCase(message.trim())) {
                        break;
                    }
                }

                // 5. Rời khỏi nhóm
                socket.leaveGroup(group);
                System.out.println("Đã rời khỏi nhóm.");

            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                if (socket != null) {
                    socket.close();
                }
            }
        }
##  IV. Lập trình Sender (Publisher - Bên gửi)
Bên gửi thì đơn giản hơn. Nó không cần "tham gia" nhóm, chỉ cần gửi gói tin đến đúng địa chỉ nhóm là được.
Các bước thực hiện:
* Tạo một MulticastSocket (không cần bind cổng).
* Lấy InetAddress của nhóm (ví dụ: "230.0.0.1").
* (Tùy chọn) Cài đặt TTL: socket.setTimeToLive(1).
* Tạo một DatagramPacket chứa dữ liệu, và đặt địa chỉ đích là địa chỉ nhóm và cổng nhóm (ví dụ: "230.0.0.1" và 8888).
* Gọi socket.send().
**Code ví dụ: MulticastSender.java (Bên gửi)**

'''java

    import java.net.DatagramPacket;
    import java.net.InetAddress;
    import java.net.MulticastSocket;
    import java.util.Scanner;

    public class MulticastSender {
        public static final String MULTICAST_GROUP = "230.0.0.1";
        public static final int PORT = 8888;

        public static void main(String[] args) {
            MulticastSocket socket = null;
            try {
                // 1. Lấy địa chỉ nhóm
                InetAddress group = InetAddress.getByName(MULTICAST_GROUP);
                
                // 2. Tạo MulticastSocket (dùng để gửi)
                socket = new MulticastSocket();
                
                // 3. (Tùy chọn) Cài đặt TTL = 1 (chỉ gửi trong mạng LAN)
                socket.setTimeToLive(1);

                System.out.println("Gõ tin nhắn để gửi đến nhóm (gõ 'exit' để thoát):");
                Scanner scanner = new Scanner(System.in);
                
                while (true) {
                    String message = scanner.nextLine();
                    byte[] buffer = message.getBytes("UTF-8");

                    // 4. Tạo packet gửi đến địa chỉ NHÓM và CỔNG
                    DatagramPacket sendPacket = new DatagramPacket(
                            buffer, 
                            buffer.length, 
                            group, // Địa chỉ nhóm
                            PORT   // Cổng nhóm
                    );

                    // 5. Gửi (giống hệt UDP)
                    socket.send(sendPacket);
                    System.out.println("Đã gửi: " + message);

                    if ("exit".equalsIgnoreCase(message.trim())) {
                        break;
                    }
                }
                
                scanner.close();

            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                if (socket != null) {
                    socket.close();
                }
            }
        }
    }