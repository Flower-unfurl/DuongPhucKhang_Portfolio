---
title: "Socket Programming Trong Java: Cánh Cửa Kết Nối Client-Server Đơn Giản"
description: "Tìm hiểu cách xây dựng ứng dụng mạng cơ bản với Socket và ServerSocket trong Java, từ chat đơn giản đến hệ thống phân tán."
date: 2025-10-22
tags: ["Java", "Network", "Socket", "Programming"]
cover: "/images/projects/ui-animations2.png"
draft: false
---

Bạn đã bao giờ tự hỏi làm thế nào các ứng dụng trao đổi dữ liệu qua mạng? Câu trả lời chính là **Socket**! Trong Java, lập trình Socket (Socket Programming) là nền tảng để xây dựng các ứng dụng mạng cơ bản, từ chat đơn giản đến các hệ thống phân tán phức tạp. Bài viết này sẽ giúp bạn hiểu rõ khái niệm Client-Server và cách tạo một kết nối cơ bản.

## Mục tiêu bài học

- Hiểu vai trò của Socket và ServerSocket trong Java
- Nắm được quy trình giao tiếp Client-Server cơ bản
- Cài đặt một Server đơn giản chấp nhận kết nối

## 1. Socket và ServerSocket: Hai Khái Niệm Cốt Lõi

Trong lập trình mạng TCP/IP của Java, chúng ta sử dụng hai loại Socket chính:

**ServerSocket**: Dùng cho Server. Nó lắng nghe các kết nối đến trên một cổng (port) cụ thể. Khi có một Client kết nối, ServerSocket sẽ tạo ra một Socket mới để giao tiếp.

**Socket**: Dùng cho cả Server (để giao tiếp sau khi kết nối được thiết lập) và Client (để khởi tạo kết nối đến Server).

## 2. Quy Trình Giao Tiếp Cơ Bản

1. **Server**: Khởi tạo ServerSocket và liên kết nó với một cổng (port)
2. **Server**: Gọi phương thức `accept()` để lắng nghe (blocking call) và chờ Client kết nối
3. **Client**: Khởi tạo Socket và cố gắng kết nối đến địa chỉ IP và cổng của Server
4. **Server**: Khi `accept()` thành công, nó trả về một đối tượng Socket mới
5. **Cả hai**: Sử dụng Input/OutputStream của Socket để gửi/nhận dữ liệu

## 3. Ví Dụ Đơn Giản: Server Echo

### Server Code

```java
import java.net.*;
import java.io.*;

public class SimpleServer {
    public static void main(String[] args) throws IOException {
        int port = 6666;
        try (ServerSocket serverSocket = new ServerSocket(port)) {
            System.out.println("Server đang lắng nghe tại port " + port);
            
            // 1. Chờ Client kết nối
            try (Socket clientSocket = serverSocket.accept();
                 PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true);
                 BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()))) {

                System.out.println("Client đã kết nối.");
                String inputLine;
                while ((inputLine = in.readLine()) != null) {
                    System.out.println("Client nói: " + inputLine);
                    out.println("Server nhận được: " + inputLine); // Gửi lại (Echo)
                    if ("bye".equalsIgnoreCase(inputLine)) break;
                }
            }
        } catch (IOException e) {
            System.out.println("Lỗi Server: " + e.getMessage());
        }
    }
}
```

### Client Code

```java
import java.net.*;
import java.io.*;

public class SimpleClient {
    public static void main(String[] args) throws IOException {
        String hostName = "localhost";
        int port = 6666;

        try (Socket socket = new Socket(hostName, port);
             PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
             BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
             BufferedReader stdIn = new BufferedReader(new InputStreamReader(System.in))) {

            String userInput;
            while ((userInput = stdIn.readLine()) != null) {
                out.println(userInput); // Gửi dữ liệu đi
                System.out.println("Server trả lời: " + in.readLine()); // Nhận phản hồi
                if ("bye".equalsIgnoreCase(userInput)) break;
            }
        } catch (UnknownHostException e) {
            System.err.println("Không tìm thấy host: " + hostName);
        } catch (IOException e) {
            System.err.println("Không thể kết nối với: " + hostName);
        }
    }
}
```

## Các từ khóa cần nhớ

- **Socket**: Điểm cuối (endpoint) của kết nối, dùng để gửi và nhận dữ liệu
- **ServerSocket**: Server lắng nghe kết nối đến trên một cổng cụ thể
- **Port**: Số hiệu dùng để xác định ứng dụng đang chạy trên máy tính
- **Blocking Call**: Phương thức `accept()` sẽ tạm dừng chương trình cho đến khi có Client kết nối

## Vận dụng

- Xây dựng các ứng dụng chat đa người dùng (cần thêm luồng - Threading)
- Phát triển các hệ thống truyền tải file đơn giản
- Làm nền tảng cho việc hiểu các giao thức HTTP, FTP...

## Tóm tắt kiến thức

Lập trình Socket trong Java sử dụng **ServerSocket** để lắng nghe và **Socket** để thiết lập kênh giao tiếp Client-Server. Đây là cách cơ bản nhất để ứng dụng Java trao đổi dữ liệu qua mạng, thường dùng giao thức TCP (đáng tin cậy).

## Nguồn tham khảo

- [Oracle Java Documentation: Socket and ServerSocket Classes](https://docs.oracle.com/javase/8/docs/api/java/net/Socket.html)
- Java Network Programming - O'Reilly