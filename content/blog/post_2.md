---
title: "Chương 2: Quản lý các luồng nhập xuất"
date: 2025-10-24
tags: ["mollis urna", "notes", "laoreet"]
categories: ["diam aliqua", "fringilla"]
description: "dictum tellus sapien vitae integer justo amet mauris cras bolestie sollicitudin dignissim"
draft: false
---

## I. Giới thiệu: Luồng (Stream) là gì?

### 1. Mục tiêu bài học
* Hiểu rõ khái niệm "luồng" (Stream) trong Java.
* Phân biệt được hai họ luồng chính: **Luồng Byte** (Byte Streams) và **Luồng Ký tự** (Character Streams).
* Nắm được các lớp cha (abstract class) cơ sở: `InputStream`, `OutputStream`, `Reader`, và `Writer`.
* Hiểu được vai trò của các luồng "bọc" (Decorator Streams) để tăng hiệu năng và chức năng.
* Biết cách sử dụng các luồng "cầu nối" (`InputStreamReader`, `OutputStreamWriter`) để chuyển đổi giữa luồng byte và luồng ký tự—một kỹ năng tối quan trọng trong lập trình mạng.

### 2. Khái niệm Stream
Trong Java, một **Stream (luồng)** là một khái niệm trừu tượng đại diện cho một **dòng chảy dữ liệu (data flow)**. Nó là một cơ chế cho phép chương trình của bạn đọc dữ liệu từ một *nguồn* (source) và ghi dữ liệu ra một *đích* (destination).

- **Nguồn (Source):** Có thể là file, bàn phím, một socket mạng, hoặc một mảng byte.
- **Đích (Destination):** Có thể là file, màn hình console, một socket mạng.

Hãy tưởng tượng Stream như một **đường ống nước một chiều**:
- **Luồng nhập (Input Stream):** "Hút" dữ liệu *từ* nguồn *vào* chương trình của bạn.
- **Luồng xuất (Output Stream):** "Đẩy" dữ liệu *từ* chương trình của bạn *ra* đích.

{{< figure src="/Blog/images/bai2.jpg" alt="Sơ đồ khái niệm Input và Output Stream" caption="Hình 1: Luồng nhập (Input) và Luồng xuất (Output)." >}}
---

## II. Phân loại các Luồng trong `java.io`

Hệ thống `java.io` có hai họ luồng chính, dựa trên loại dữ liệu mà chúng xử lý:

### 1. Luồng Byte (Byte Streams)
- **Xử lý:** Dữ liệu nhị phân thô (raw binary data), từng **byte** một (8-bit).
- **Lớp cha:** `InputStream` và `OutputStream`.
- **Khi nào dùng:** Khi làm việc với *bất kỳ loại dữ liệu nào*. Đặc biệt hữu ích cho file hình ảnh, file âm thanh, file `.exe`, và **dữ liệu mạng thô từ Socket**.
- **Lớp cơ sở phổ biến:** `FileInputStream`, `FileOutputStream`, và luồng lấy từ Socket: `socket.getInputStream()`.

### 2. Luồng Ký tự (Character Streams)
- **Xử lý:** Dữ liệu văn bản (text), từng **ký tự** một (16-bit Unicode).
- **Lớp cha:** `Reader` và `Writer`.
- **Khi nào dùng:** Chỉ dùng khi làm việc với dữ liệu văn bản (text). Chúng có khả năng xử lý các bộ mã hóa ký tự (character encoding) như `UTF-8`, `ASCII`...
- **Lớp cơ sở phổ biến:** `FileReader`, `FileWriter`.

{{< figure src="/Blog/images/bai2-1.webp" alt="Cây phân cấp các lớp java.io" caption="Hình 2: Phân cấp 4 họ luồng chính trong java.io (InputStream, OutputStream, Reader, Writer)." >}}

---

## III. Các luồng "Bọc" và "Cầu nối" (Decorators & Bridges)

Hiếm khi chúng ta dùng các luồng cơ sở (như `FileInputStream`) một mình. Thay vào đó, chúng ta "bọc" (wrap) chúng bằng các luồng khác để thêm chức năng. Đây là một mẫu thiết kế (design pattern) gọi là **Decorator**.

### 1. Luồng đệm (Buffered Streams)
- **Các lớp:** `BufferedInputStream`, `BufferedOutputStream`, `BufferedReader`, `BufferedWriter`.
- **Tại sao dùng?** Vì tốc độ! Đọc/ghi từng byte một từ ổ đĩa hoặc mạng rất chậm. Luồng đệm tạo ra một bộ nhớ tạm (buffer), nó đọc/ghi một **khối dữ liệu lớn** (ví dụ: 8KB) vào buffer, sau đó chương trình của bạn chỉ cần đọc/ghi từ buffer rất nhanh.
- **`BufferedReader`** đặc biệt hữu ích vì nó có phương thức `readLine()` để đọc từng dòng text.

### 2. Luồng Cầu nối (Bridge Streams) - Cực kỳ quan trọng!
Đây là mấu chốt kết nối giữa Lập trình mạng và I/O:

- **Vấn đề:** Mạng (Socket) chỉ cung cấp **Luồng Byte** (`InputStream`, `OutputStream`). Nhưng ứng dụng chat, web... lại muốn gửi/nhận **Luồng Ký tự** (text).
- **Giải pháp:**
    - `InputStreamReader`: Là một `Reader` "bọc" một `InputStream`. Nó "cầu nối" bằng cách **đọc các byte thô** từ `InputStream` và **giải mã chúng thành các ký tự** (ví dụ: dùng UTF-8).
    - `OutputStreamWriter`: Là một `Writer` "bọc" một `OutputStream`. Nó "cầu nối" bằng cách **nhận các ký tự** và **mã hóa chúng thành các byte** (ví dụ: dùng UTF-8) để gửi đi qua `OutputStream`.

---

## IV. Kịch bản I/O phổ biến trong Lập trình mạng (Java)

Hãy xem cách kết hợp các luồng này để xây dựng một ứng dụng chat đơn giản (gửi/nhận text). Giả sử bạn đã có một đối tượng `Socket` tên là `clientSocket`.

```markdown
### 1. KỊCH BẢN 1: Đọc văn bản (text) từ Client gửi đến Server
Chúng ta muốn đọc từng dòng (ví dụ: `readLine()`).

```java
/* Mục tiêu: Đọc text hiệu quả (có buffer, đọc theo dòng) từ một Socket.
 *
 * Sơ đồ "bọc" luồng:
 * Socket -> InputStream (byte) -> InputStreamReader (char) -> BufferedReader (char + buffer)
 */

// 1. Lấy luồng byte thô từ socket
InputStream inputStream = clientSocket.getInputStream();

// 2. Tạo "cầu nối" để chuyển byte thành ký tự (Chỉ định rõ UTF-8)
Reader reader = new InputStreamReader(inputStream, "UTF-8");

// 3. "Bọc" bằng BufferedReader để đọc hiệu quả và có hàm readLine()
BufferedReader bufferedReader = new BufferedReader(reader);

// 4. Sẵn sàng đọc dữ liệu
String messageFromClient = bufferedReader.readLine();
System.out.println("Client nói: " + messageFromClient);

/* Mục tiêu: Gửi text hiệu quả (có buffer, gửi theo dòng) tới một Socket.
 *
 * Sơ đồ "bọc" luồng:
 * Socket -> OutputStream (byte) -> OutputStreamWriter (char) -> PrintWriter (char + buffer + tiện ích)
 */
```
```markdown
### 2. KỊCH BẢN 2: Gửi văn bản (text) từ Server đến Client
```java
/* Mục tiêu: Gửi text hiệu quả (có buffer, gửi theo dòng) tới một Socket.
 *
 * Sơ đồ "bọc" luồng:
 * Socket -> OutputStream (byte) -> OutputStreamWriter (char) -> PrintWriter (char + buffer + tiện ích)
 */

// 1. Lấy luồng byte thô ra socket
OutputStream outputStream = clientSocket.getOutputStream();

// 2. Tạo "cầu nối" để chuyển ký tự (chúng ta gõ) thành byte (để gửi đi)
Writer writer = new OutputStreamWriter(outputStream, "UTF-8");

// 3. (Khuyến nghị) Dùng PrintWriter: Đây là lớp bọc tiện lợi nhất.
// Nó cung cấp hàm .println() quen thuộc.
// Tham số 'true' (autoFlush) nghĩa là nó sẽ tự động "đẩy" (flush) dữ liệu đi
// ngay khi chúng ta gọi println(), rất quan trọng trong mạng.
PrintWriter printWriter = new PrintWriter(writer, true);

// 4. Sẵn sàng gửi dữ liệu
printWriter.println("Chào client, tôi là Server!");