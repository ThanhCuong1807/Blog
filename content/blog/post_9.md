---
title: "Chương 9: Phân tán đối tượng trong java bằng rmi"
date: 2025-10-24
tags: ["mollis urna", "notes", "laoreet"]
categories: ["diam aliqua", "fringilla"]
description: "dictum tellus sapien vitae integer justo amet mauris cras bolestie sollicitudin dignissim"
draft: false
---

## I. Giới thiệu: RMI (Remote Method Invocation) là gì?
### 1. Mục tiêu bài học
* Hiểu rõ khái niệm RMI (Gọi phương thức từ xa) và tại sao nó là một cấp độ trừu tượng cao hơn Socket.
* Phân biệt được sự khác nhau giữa Socket (gửi dữ liệu) và RMI (gọi hành vi/phương thức).
* Nắm vững các thành phần cốt lõi của RMI: Remote Interface, Implementation, RMI Registry, Server và Client.
* Xây dựng được một ứng dụng Client-Server hoàn chỉnh sử dụng RMI (ví dụ: một máy tính từ xa).
### 2. So sánh Socket và RMI
Hãy tưởng tượng bạn là một giám đốc (Client) và nhân viên (Server) ở một văn phòng xa.
* Lập trình Socket (Bài 5-8): Giống như bạn viết một lá thư (gửi String hoặc Object) cho nhân viên, nói rằng: "Hãy tính tổng 5 và 10, rồi gửi thư kết quả lại cho tôi". Bạn phải tự định nghĩa cách "viết thư" và "đọc thư".
* Lập trình RMI (Bài 9): Giống như bạn nhấc điện thoại lên và gọi trực tiếp cho phương thức tinhTong(5, 10) của nhân viên đó. Bạn không cần quan tâm lá thư được gửi thế nào, RMI lo hết. Bạn chỉ cần gọi hàm.  

**RMI (Remote Method Invocation)** là một API của Java cho phép một đối tượng trên một máy ảo Java (Client) gọi các phương thức của một đối tượng trên một máy ảo Java khác (Server).
{{< figure src="/Blog/images/bai9.webp" alt="Sơ đồ kiến trúc tổng quan của Java RMI" caption="Hình 1: Client gọi một phương thức trên 'Stub' (proxy), RMI lo việc giao tiếp mạng đến Server." >}}
## II. Các thành phần của RMI
Một ứng dụng RMI cơ bản có 5 thành phần:
* Remote Interface (Giao diện từ xa): Một file .java định nghĩa các phương thức mà Client có thể gọi. Nó phải extends java.rmi.Remote.
* Implementation (Lớp triển khai): Một file .java (phía Server) triển khai (implements) Remote Interface. Nó chứa code logic thật sự của các phương thức. Nó phải extends UnicastRemoteObject.
* RMI Registry (Bộ đăng ký): Giống như "Danh bạ điện thoại". Server sẽ "đăng ký" dịch vụ của mình với Registry (ví dụ: "Dịch vụ tính toán" ở cổng 1099).
* Server (Máy chủ): Một chương trình main() tạo ra một đối tượng Implementation và "đăng ký" (bind) nó với RMI Registry.
* Client (Máy khách): Một chương trình main() khác "tra cứu" (lookup) dịch vụ trong RMI Registry, nhận về một đối tượng "Stub", và gọi phương thức trên Stub đó.
## III. Lập trình Ví dụ: Máy tính từ xa
Chúng ta sẽ xây dựng một dịch vụ cho phép Client gọi hàm add(a, b) từ Server.  
**Bước 1:** Tạo Remote Interface (ICalculator.java)  
Đây là "hợp đồng". Tất cả các phương thức phải throws RemoteException.

'''java

    import java.rmi.Remote;
    import java.rmi.RemoteException;

    // 1. Phải extends Remote
    public interface ICalculator extends Remote {
        
        // 2. Tất cả phương thức client gọi phải throws RemoteException
        int add(int a, int b) throws RemoteException;
        int subtract(int a, int b) throws RemoteException;
    }
**Bước 2:** Tạo Lớp triển khai (CalculatorImpl.java)  
Đây là code logic thật sự ở Server.  

'''java

    import java.rmi.server.UnicastRemoteObject;
    import java.rmi.RemoteException;

    // 1. Implements Interface
    // 2. Extends UnicastRemoteObject để "lắng nghe" các lời gọi từ xa
    public class CalculatorImpl extends UnicastRemoteObject implements ICalculator {

        // 3. Phải có một constructor ném ra RemoteException
        public CalculatorImpl() throws RemoteException {
            super(); // Gọi constructor của cha
        }

        // 4. Triển khai logic thật sự
        @Override
        public int add(int a, int b) throws RemoteException {
            System.out.println("Server đang thực hiện phép cộng: " + a + " + " + b);
            return a + b;
        }

        @Override
        public int subtract(int a, int b) throws RemoteException {
            System.out.println("Server đang thực hiện phép trừ: " + a + " - " + b);
            return a - b;
        }
    }
**Bước 3:** Tạo Server Host (RmiServer.java)  
Chương trình này tạo Registry và đăng ký dịch vụ.  

'''java

    import java.rmi.registry.LocateRegistry;
    import java.rmi.registry.Registry;

    public class RmiServer {
        public static final int RMI_PORT = 1099; // Cổng chuẩn của RMI Registry
        public static final String SERVICE_NAME = "CalculatorService"; // Tên dịch vụ

        public static void main(String[] args) {
            try {
                // 1. Tạo đối tượng triển khai
                CalculatorImpl calculator = new CalculatorImpl();

                // 2. Tạo một RMI Registry (danh bạ) tại cổng 1099
                Registry registry = LocateRegistry.createRegistry(RMI_PORT);
                
                // 3. "Đăng ký" (bind) dịch vụ với một cái tên
                // Client sẽ dùng tên này để tìm
                registry.bind(SERVICE_NAME, calculator);

                System.out.println("RMI Server đang chạy tại cổng " + RMI_PORT + "...");
                System.out.println("Đã đăng ký dịch vụ: " + SERVICE_NAME);

            } catch (Exception e) {
                System.err.println("Server thất bại: " + e.getMessage());
                e.printStackTrace();
            }
        }
    }
**Bước 4:** Tạo Client (RmiClient.java)  
Chương trình này tìm dịch vụ và gọi phương thức.  

'''java

    import java.rmi.registry.LocateRegistry;
    import java.rmi.registry.Registry;

    public class RmiClient {
        public static final String SERVER_HOST = "localhost"; // Địa chỉ Server
        public static final int RMI_PORT = 1099;
        public static final String SERVICE_NAME = "CalculatorService";

        public static void main(String[] args) {
            try {
                // 1. Lấy Registry (danh bạ) đang chạy tại host/port
                Registry registry = LocateRegistry.getRegistry(SERVER_HOST, RMI_PORT);

                // 2. "Tra cứu" (lookup) dịch vụ bằng tên
                // RMI trả về một đối tượng "Stub" (proxy)
                // Phải ép kiểu về đúng Interface
                ICalculator calculator = (ICalculator) registry.lookup(SERVICE_NAME);

                System.out.println("Đã kết nối tới RMI Server.");
                
                // 3. Gọi phương thức như thể nó ở local
                int sum = calculator.add(10, 5);
                int diff = calculator.subtract(10, 5);

                System.out.println("Kết quả từ Server (add 10+5): " + sum);
                System.out.println("Kết quả từ Server (sub 10-5): " + diff);

            } catch (Exception e) {
                System.err.println("Client thất bại: " + e.getMessage());
                e.printStackTrace();
            }
        }
    }
