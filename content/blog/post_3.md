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
* Giải thích được tại sao lập trình mạng **bắt buộc** phải sửS dụng đa tuyến.
* Nắm được hai cách tạo Thread trong Java: Kế thừa lớp `Thread` và triển khai `Runnable` (Interface).
* Áp dụng đa tuyến để xây dựng một **Server** hoàn chỉnh có khả năng xử lý nhiều Client cùng lúc mà không bị "khóa".

### 2. Vấn đề "Blocking"
Ở Bài 2, chúng ta đã học về Kịch bản 1 (Đọc) và Kịch bản 2 (Ghi). Nếu bạn viết chúng nối tiếp nhau trong một Server, bạn sẽ gặp một vấn đề nghiêm trọng:

```java
// Mã giả của Server đơn tuyến (SAI)
serverSocket.accept(); // Chấp nhận Client 1

// Bắt đầu Kịch bản 1 (Đọc)
String msg = bufferedReader.readLine(); // <-- CHƯƠNG TRÌNH DỪNG LẠI Ở ĐÂY

// Kịch bản 2 (Ghi) sẽ không bao giờ chạy
// cho đến khi Client 1 gửi tin nhắn.
printWriter.println("Gửi tin nhắn");

// Server không thể chấp nhận Client 2
// vì đang bận chờ Client 1.
serverSocket.accept(); // <-- Dòng này không bao giờ tới