+++
author = "Darshan Baral"
title = "Chương 1: Tổng quan về lập trình mạng"
date = "2025-10-24"
description = "Sample article showcasing formatting for HTML elements."
tags = [
    "markdown",
    "css",
    "html",
    "themes",
]
categories = [
    "themes",
    "syntax",
]
+++
## I. Giới thiệu chung
### 1. Mục tiêu bài học
- Định nghĩa chính xác "Lập trình mạng" (Network Programming) là gì và nêu được tầm quan trọng của nó.
- Hiểu và phân biệt được các mô hình kiến trúc mạng phổ biến: Client-Server và Peer-to-Peer (P2P).
- Giải thích rõ ràng các khái niệm nền tảng: Giao thức (Protocol), Địa chỉ IP (IP Address) và Số hiệu cổng (Port Number).
- Phân biệt sâu sắc hai giao thức vận chuyển cốt lõi: TCP và UDP về đặc điểm, độ tin cậy và trường hợp sử dụng.
- Nắm bắt được khái niệm "Socket" và vai trò của nó như là một giao diện lập trình ứng dụng (API) cho giao tiếp mạng.
### 2. Lập trình mạng là gì?
- Lập trình mạng là quá trình viết các chương trình máy tính (phần mềm, ứng dụng) cho phép chúng giao tiếp và trao đổi dữ liệu với nhau qua một mạng máy tính (như mạng LAN nội bộ hoặc mạng Internet toàn cầu).
{{< figure src="bai1.webp" alt="Lập trình mạng là gì" caption="Lập trình mạng" >}}
- Về bản chất, nó không phải là lập trình ra mạng, mà là lập trình trên mạng. Chúng ta sử dụng các cơ sở hạ tầng mạng (dây cáp, router, switch) đã có sẵn để xây dựng các ứng dụng có khảg năng "nói chuyện" với nhau, bất kể chúng đang chạy trên cùng một máy hay ở hai đầu đối diện của thế giới.
- Ví dụ thực tế: Trình duyệt web của bạn (client) yêu cầu và nhận dữ liệu từ một máy chủ web (server) để hiển thị trang web. Ứng dụng Zalo/Messenger gửi tin nhắn từ điện thoại của bạn đến điện thoại của bạn bè.
## II. Mô hình Kiến trúc và Giao thức
### 1. Mô hình tham chiếu OSI và TCP/IP
Để các máy tính có thể giao tiếp, chúng cần tuân theo một bộ quy tắc. Các mô hình OSI và TCP/IP mô tả các tầng (lớp) khác nhau của giao tiếp mạng.
- Mô hình OSI (7 lớp): Là mô hình lý thuyết, chi tiết.
- Mô hình TCP/IP (4 hoặc 5 lớp): Là mô hình thực tế đang được sử dụng trên Internet.
{{< figure src="bai1.2.jpg" alt="Lập trình mạng là gì" caption="Mô hình OSI và TCP/CP" >}}
Với tư cách là lập trình viên mạng, chúng ta chủ yếu làm việc ở hai tầng trên cùng:
- Tầng Ứng dụng (Application Layer): Nơi các giao thức quen thuộc như HTTP (Web), FTP (File), SMTP (Mail) hoạt động.
- Tầng Vận chuyển (Transport Layer): Nơi hai giao thức cốt lõi TCP và UDP hoạt động. Đây là tầng chúng ta tương tác trực tiếp nhiều nhất khi lập trình socket.
### 2. Mô hình ứng dụng Client-Server (Khách - Chủ)
- Server (Máy chủ): Là một chương trình "bị động", luôn chạy trên một máy tính (thường là máy chủ mạnh), mở một cổng (Port) cụ thể và "lắng nghe" các yêu cầu kết nối từ Client. Nhiệm vụ của nó là cung cấp dịch vụ (ví dụ: cung cấp trang web, dữ liệu database, xử lý logic...).
- Client (Máy khách): Là một chương trình "chủ động", chạy trên máy của người dùng (PC, điện thoại). Nó chủ động gửi yêu cầu (Request) kết nối đến địa chỉ IP và Cổng của Server. Sau khi gửi yêu cầu, nó chờ Server xử lý và gửi lại kết quả (Response).
{{< figure src="bai1.3.jpg" alt="Lập trình mạng là gì" caption="Mô hình Client-Server" >}}
Ví dụ: Web (Trình duyệt là Client, Web Server là Server), Game Online (Ứng dụng game là Client, Máy chủ game là Server).
### 3. Mô hình Peer-to-Peer (P2P - Mạng ngang hàng)
- Trong mô hình này, không có máy chủ trung tâm. Mọi máy tính trong mạng (gọi là "Peer") đều có vai trò và chức năng như nhau. Một máy vừa có thể là Client (khi nó yêu cầu tài nguyên từ máy khác) vừa có thể là Server (khi nó cung cấp tài nguyên cho máy khác).
- Ví dụ: BitTorrent (chia sẻ file), các ứng dụng chat/gọi video phi tập trung, tiền mã hóa (Blockchain).
## III. Các thành phần định danh trong mạng
### 1. Địa chỉ IP (IP Address)
- Là gì: Là một địa chỉ logic, duy nhất để định danh một thiết bị (máy tính, router, điện thoại) trên mạng.
- Vai trò: Giống như "địa chỉ nhà" (số nhà, tên đường, quận, thành phố) của bạn. Nó cho phép các gói tin biết phải đi đến máy tính nào.
{{< figure src="IP.jpg" alt="" caption="Mô hình địa chỉ IP" >}}
- Phiên bản:  
    IPv4: (Vd: 172.217.14.228) Dạng 32-bit, đang dần cạn kiệt.  
    IPv6: (Vd: 2001:0db8:85a3:0000:0000:8a2e:0370:7334) Dạng 128-bit, cung cấp không gian địa chỉ khổng lồ.
### 2. Số hiệu cổng (Port Number)  
Vấn đề: Một máy tính (cùng 1 địa chỉ IP) có thể chạy rất nhiều ứng dụng mạng cùng lúc (vừa duyệt web, vừa chat, vừa nghe nhạc online...). Làm sao để gói tin đến đúng ứng dụng?
- Là gì: Port là một con số (nguyên 16-bit, từ 0 đến 65535) dùng để định danh một ứng dụng cụ thể đang chạy trên máy đó.
- Vai trò: Giống như "số phòng" trong một tòa nhà chung cư (tòa nhà là địa chỉ IP).
{{< figure src="bai1.4.png" alt="Lập trình mạng là gì" caption="Mô hình Post" >}}
- Phân loại:  
    Cổng Nổi tiếng (Well-known Ports): 0 - 1023. Dành riêng cho các dịch vụ chuẩn. (Vd: HTTP dùng cổng 80, HTTPS dùng cổng 443, FTP dùng cổng 21).  
    Cổng Đã đăng ký (Registered Ports): 1024 - 49151. Dành cho các ứng dụng, dịch vụ cụ thể của các công ty.  
    Cổng Động (Dynamic/Private Ports): 49152 - 65535. Dùng cho các kết nối tạm thời (ví dụ: trình duyệt của bạn khi kết nối đến server sẽ được cấp 1 cổng động ngẫu nhiên).
## IV. Giao thức Tầng Vận chuyển: TCP vs. UDP
Khi hai ứng dụng quyết định "nói chuyện", chúng phải chọn một "phương thức vận chuyển" dữ liệu. TCP và UDP là hai lựa chọn phổ biến nhất.
### 1. TCP (Transmission Control Protocol - Giao thức điều khiển truyền vận)
- Đặc điểm: Hướng kết nối (Connection-Oriented).
- Mô tả: Giống như một cuộc gọi điện thoại. Hai bên phải nhấc máy, chào hỏi và thiết lập một đường truyền ổn định (gọi là "bắt tay 3 bước" - 3-way handshake) trước khi bắt đầu nói chuyện (truyền dữ liệu).
- Độ tin cậy: Rất cao:  
  Đảm bảo dữ liệu đến nơi, không mất mát (nếu mất sẽ tự động gửi lại).  
  Đảm bảo dữ liệu đến đúng thứ tự (tự động sắp xếp lại các gói tin nếu bị xáo trộn).  
  Có cơ chế kiểm soát luồng (flow control) và kiểm soát tắc nghẽn (congestion control).
- Tốc độ: Chậm hơn UDP vì phải tốn chi phí (overhead) để quản lý kết nối và đảm bảo độ tin cậy.
- Khi nào dùng: Khi độ chính xác của dữ liệu là tuyệt đối quan trọng: Duyệt web (HTTP/HTTPS), Gửi/nhận Email (SMTP/POP3), Truyền file (FTP).
### 2. UDP (User Datagram Protocol - Giao thức Dữ liệu người dùng)
- Đặc điểm: Phi kết nối (Connectionless).
- Mô tả: Giống như gửi bưu thiếp. Bạn chỉ cần ghi địa chỉ người nhận và "ném" nó vào hòm thư. Bạn không biết khi nào nó đến, nó có đến hay không, hay nó có đến cùng lúc với các bưu thiếp khác hay không.
- Độ tin cậy: Thấp (Không tin cậy).  
  "Nỗ lực tối đa" (Best-effort): Cố gắng gửi nhưng không đảm bảo gì cả.  
  Dữ liệu có thể bị mất (lost), trùng lặp (duplicated), hoặc đến sai thứ tự (out-of-order).  
- Tốc độ: Rất nhanh. Gần như không có chi phí quản lý, chỉ đơn giản là "gói và gửi".  
  Khi nào dùng: Khi tốc độ là quan trọng nhất và việc mất một vài gói tin có thể chấp nhận được: Game online (cần cập nhật vị trí nhanh), Livestream video/audio, Gọi thoại/video (VoIP), Phân giải tên miền (DNS).
#### Bảng so sánh TCP vs UDP

| Đặc điểm     | TCP (Transmission Control Protocol)  | UDP (User Datagram Protocol) |
| ---------- | --------- | ----------------- | 
| Kiểu kết nối  | Hướng kết nối (Connection-Oriented)  | Phi kết nối (Connectionless) |
| Mô tả  | Giống như một cuộc gọi điện thoại. Phải "bắt tay 3 bước" (3-way handshake) để thiết lập kết nối trước khi truyền dữ liệu.  | Giống như gửi bưu thiếp. Gói và gửi đi ngay lập tức, không cần thiết lập kết nối. |
| Độ tin cậy  | Cao (Tin cậy)  | Thấp (Không tin cậy) |
| Đảm bảo  | Đảm bảo dữ liệu đến nơi, không mất mát, không trùng lặp. Nếu mất, sẽ tự động gửi lại.  | "Nỗ lực tối đa" (Best-effort). Dữ liệu có thể bị mất, lặp, hoặc đến sai thứ tự mà không báo trước. |
| Thứ tự gói tin | Đảm bảo đúng thứ tự. Các gói tin được đánh số và sắp xếp lại ở nơi nhận.  | Không đảm bảo thứ tự. Gói nào đến trước được xử lý trước. |
| Kiểm soát lỗi  | Có. Sử dụng checksum và acknowledgments (ACK) để kiểm tra lỗi và xác nhận đã nhận.  | Có (Cơ bản). Chỉ dùng checksum để kiểm tra lỗi cơ bản của gói tin, không có cơ chế báo nhận hay gửi lại. |
| Kiểm soát luồng  | Có. Có cơ chế kiểm soát luồng (flow control) và kiểm soát tắc nghẽn (congestion control) để tránh làm quá tải mạng hoặc máy nhận.  | Không. Gửi đi liên tục mà không quan tâm máy nhận có xử lý kịp hay không. |
| Tốc độ | Chậm hơn  | Rất nhanh |
| Chi phí (Overhead) | Cao. Phần header của gói tin lớn hơn (tối thiểu 20 bytes) do phải chứa nhiều thông tin quản lý (số thứ tự, cờ, ACK...). |Thấp. Phần header rất nhỏ (chỉ 8 bytes), tốn ít tài nguyên xử lý. |
| Ví dụ sử dụng  | 🌐Duyệt web (HTTP/HTTPS) 📧 Gửi/nhận Email (SMTP, POP3) 📁 Truyền file (FTP)  | 🎮 Game online 📺 Livestream video/audio 🗣️ Gọi thoại/video (VoIP) 📡 Phân giải tên miền (DNS) |
## V. Socket: Cửa ngõ để Lập trình mạng
### 1. Socket là gì?
- Định nghĩa lý thuyết: Một Socket là một "điểm cuối" (endpoint) của một kênh giao tiếp hai chiều. Một kênh giao tiếp hoàn chỉnh được xác định bởi một cặp Socket (Socket phía Client và Socket phía Server).
- Định nghĩa thực tế: Một Socket được xác định duy nhất bởi sự kết hợp của một Địa chỉ IP và một Số hiệu Cổng (Ví dụ: 172.217.14.228:80).
- Định nghĩa Lập trình: Đối với lập trình viên, Socket là một Giao diện Lập trình Ứng dụng (API). Nó là một đối tượng (object) hoặc một thư viện mà hệ điều hành cung cấp, cho phép chúng ta:  
  Tạo kết nối (phía Client).  
  Lắng nghe kết nối (phía Server).  
  Gửi dữ liệu.  
  Nhận dữ liệu.
- Nói đơn giản, Socket là "cái ổ cắm" mà lập trình viên chúng ta "cắm phích" (chương trình của mình) vào để sử dụng mạng.
### 2. Phân loại Socket (tương ứng với TCP/UDP)
- Stream Sockets (Socket luồng): Sử dụng giao thức TCP. Cung cấp một kênh giao tiếp tin cậy, có thứ tự, hai chiều. Dữ liệu được đọc/ghi như một "luồng" (stream) liên tục.  
 Trong Java: Socket và ServerSocket.
- Datagram Sockets (Socket dữ liệu): Sử dụng giao thức UDP. Cung cấp một kênh giao tiếp không tin cậy. Dữ liệu được gửi/nhận dưới dạng các "gói" (datagram) riêng lẻ, độc lập.  
  Trong Java: DatagramSocket và DatagramPacket.