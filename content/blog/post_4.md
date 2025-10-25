---
title: "Chương 4: Quản lý địa chỉ kết nối mạng"
date: 2025-10-24
tags: ["mollis urna", "notes", "laoreet"]
categories: ["diam aliqua", "fringilla"]
description: "dictum tellus sapien vitae integer justo amet mauris cras bolestie sollicitudin dignissim"
draft: false
---
## I. Giới thiệu
### 1. Mục tiêu bài học
* Hiểu được tại sao cần các lớp Java để biểu diễn địa chỉ IP thay vì chỉ dùng chuỗi (String).
* Nắm vững cách sử dụng lớp InetAddress để làm việc với địa chỉ IP và tên miền (DNS lookup).
* Biết cách lấy địa chỉ IP của máy local (localhost) và của một tên miền cụ thể (ví dụ: google.com).
* Hiểu rõ vai trò của lớp InetSocketAddress là sự kết hợp của IP và số hiệu cổng (Port).
* Sẵn sàng sử dụng các đối tượng địa chỉ này cho Bài 5 (Lập trình Socket).
### 2. Tại sao cần lớp quản lý địa chỉ?
* Trong mạng, một kết nối không chỉ cần "tên" (như google.com) mà cần địa chỉ cụ thể (như 142.250.204.142). Java cung cấp các lớp để quản lý thông tin này.
* Thay vì truyền các chuỗi String ("142.250.204.142") đi khắp nơi, chúng ta sẽ dùng các đối tượng như InetAddress. Các đối tượng này chứa nhiều thông tin hơn (như tên máy, địa chỉ IP) và có các phương thức hữu ích (như kiểm tra kết nối).
## II. Lớp InetAddress - Biểu diễn một địa chỉ IP
* Lớp InetAddress là đại diện của một địa chỉ IP (cả IPv4 và IPv6).
**Lưu ý quan trọng**: Lớp này không có constructor public. Bạn không thể tạo mới nó bằng new InetAddress(). Thay vào đó, bạn phải dùng các phương thức "factory" (phương thức static trả về một đối tượng) để lấy một thực thể.
{{< figure src="/Blog/images/bai4-1.png" alt="Sơ đồ minh họa quá trình DNS Lookup" caption="Hình 1: Quá trình máy tính hỏi DNS Server 'https://www.google.com/search?q=google.com là gì?' và nhận lại địa chỉ IP." >}}
### 1. Lấy địa chỉ IP từ Tên miền (DNS Lookup)
Đây là cách phổ biến nhất, dùng getByName(). Java sẽ tự động thực hiện một truy vấn DNS (Domain Name System) để tìm IP tương ứng với tên miền bạn cung cấp.
'''java

    import java.net.InetAddress;
    import java.net.UnknownHostException;

    public class TestInetAddress {
        public static void main(String[] args) {
            try {
                // 1. Lấy địa chỉ IP của một tên miền
                InetAddress googleAddress = InetAddress.getByName("google.com");
                System.out.println("Địa chỉ IP Google: " + googleAddress.getHostAddress());
                System.out.println("Tên máy Google: " + googleAddress.getHostName());

                // 2. Lấy địa chỉ IP của máy local (máy của bạn)
                InetAddress localAddress = InetAddress.getLocalHost();
                System.out.println("\nĐịa chỉ IP Local: " + localAddress.getHostAddress());
                System.out.println("Tên máy Local: " + localAddress.getHostName());

            } catch (UnknownHostException e) {
                System.err.println("Không thể tìm thấy máy chủ: " + e.getMessage());
            }
        }
    }
### 2. Lấy tất cả địa chỉ IP của một tên miền
Một tên miền lớn (như google.com hay facebook.com) có thể trỏ đến nhiều địa chỉ IP khác nhau để cân bằng tải. Bạn có thể lấy tất cả bằng getAllByName().
'''java
    import java.net.InetAddress;
    import java.net.UnknownHostException;

    public class TestAllIPs {
        public static void main(String[] args) {
            try {
                // Lấy TẤT CẢ các địa chỉ IP của google.com
                InetAddress[] googleAddresses = InetAddress.getAllByName("google.com");
            
                System.out.println("Các địa chỉ IP của Google:");
                for (InetAddress addr : googleAddresses) {
                    System.out.println("- " + addr.getHostAddress());
                }

            } catch (UnknownHostException e) {
                System.err.println("Không thể tìm thấy máy chủ: " + e.getMessage());
            }
        }
    }
## III. Các phương thức hữu ích khác của InetAddress
InetAddress cung cấp một số phương thức tiện ích khác, ví dụ như kiểm tra khả năng kết nối (tương tự lệnh ping).
**Kiểm tra kết nối (isReachable)**
Phương thức isReachable(timeout) cố gắng kết nối đến máy chủ trong một khoảng thời gian (miligiây).
'''java
    import java.net.InetAddress;

    public class TestReachable {
        public static void main(String[] args) {
            try {
                InetAddress host = InetAddress.getByName("google.com");
                int timeOut = 3000; // Thời gian chờ là 3 giây (3000ms)

                // Kiểm tra xem có thể "ping" tới máy chủ không
                boolean isReachable = host.isReachable(timeOut);

                if (isReachable) {
                    System.out.println(host.getHostName() + " có thể kết nối!");
                } else {
                    System.out.println(host.getHostName() + " không thể kết nối.");
                }

            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
**Lưu ý:** isReachable() không phải lúc nào cũng chính xác 100% vì một số máy chủ có thể chặn (block) các yêu cầu ping (ICMP packets).
## IV. Lớp InetSocketAddress - (IP + Port)
Như chúng ta đã học ở Bài 1, một kết nối mạng hoàn chỉnh cần cả Địa chỉ IP và Số hiệu Cổng (Port).
* InetAddress = Chỉ là địa chỉ IP (Giống "Địa chỉ tòa nhà").
* InetSocketAddress = Địa chỉ IP + Port (Giống "Phòng số 80 tại tòa nhà đó").
* Lớp InetSocketAddress đóng gói cả hai thông tin này vào một đối tượng duy nhất. Nó có constructor public (new()).
'''java
    import java.net.InetAddress;
    import java.net.InetSocketAddress;

    public class TestSocketAddress {
        public static void main(String[] args) {
            try {
                // Cách 1: Tạo từ tên miền và cổng
                // Java sẽ tự động tra cứu DNS
                InetSocketAddress socketAddr1 = new InetSocketAddress("google.com", 80);

                // Cách 2: Tạo từ đối tượng InetAddress và cổng
                InetAddress ip = InetAddress.getByName("google.com");
                InetSocketAddress socketAddr2 = new InetSocketAddress(ip, 80);

                System.out.println("Địa chỉ Socket 1: " + socketAddr1);
                System.out.println("Địa chỉ Socket 2: " + socketAddr2);

                // Lấy thông tin ra
                System.out.println("\nTên máy: " + socketAddr1.getHostName());
                System.out.println("Địa chỉ IP: " + socketAddr1.getAddress().getHostAddress());
                System.out.println("Cổng: " + socketAddr1.getPort());

            } catch (Exception e) {
                e.printStackTrace();
            }
        }   
    }