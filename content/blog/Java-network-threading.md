---
title1: "Java-network-threading" 
title: "Xử lý đa luồng trong lập trình mạng Java"
description: "Ứng dụng Server trong thế giới thực cần phải phục vụ hàng trăm, thậm chí hàng nghìn, kết nối đồng thời. Đây là lúc Multi-Threading trở nên cực kỳ quan trọng."
date: 2025-10-23
tags: ["Java", "Network", "Threading", "Socket", "Server"]
cover: "/images/blog/java-network-threading.png"
draft: false
---

Ứng dụng Server trong thế giới thực cần phải phục vụ hàng trăm, thậm chí hàng nghìn, kết nối đồng thời. Nếu bạn chạy một Server đơn luồng (Single-Threaded), nó sẽ **bị treo (block)** khi chờ một Client nào đó gửi dữ liệu. Đây là lúc **Multi-Threading** (Đa luồng) trong Java trở nên cực kỳ quan trọng đối với lập trình mạng.

## Mục tiêu bài học

- Hiểu vấn đề của Server đơn luồng trong lập trình mạng.
- Áp dụng **Thread** để xử lý nhiều kết nối Client đồng thời.
- Tìm hiểu về giao tiếp giữa các luồng.

## 1. Vấn Đề của Socket Blocked I/O

Như đã thấy ở bài trước, khi Server gọi `serverSocket.accept()` hoặc các phương thức đọc/ghi dữ liệu của Socket (`in.readLine()`), luồng hiện tại sẽ **ngừng lại** (block) cho đến khi có sự kiện xảy ra.
* Nếu **Server đơn luồng**, khi đang xử lý Client A, nó không thể chấp nhận kết nối mới hoặc xử lý dữ liệu từ Client B. Hệ thống sẽ **bị tắc nghẽn**.

## 2. Giải Pháp: Multi-Threading Cho Mỗi Client

Giải pháp tiêu chuẩn là gán một **Luồng (Thread)** riêng biệt cho **mỗi Client** được kết nối thành công.

**Quy trình:**
1.  Luồng chính (Main Thread) chạy **ServerSocket** và liên tục gọi `accept()`.
2.  Mỗi lần `accept()` thành công (có Client mới), Luồng chính sẽ tạo ra một **Luồng xử lý mới** (`ClientHandler Thread`).
3.  Luồng xử lý mới này sẽ chịu trách nhiệm **toàn bộ giao tiếp** với Client đó (đọc/ghi dữ liệu) mà không làm ảnh hưởng đến Luồng chính hay các Luồng xử lý Client khác.

## 3. Cài Đặt Client Handler Thread

Bạn cần tạo một Class mới, ví dụ `ClientHandler`, thực thi giao diện **`Runnable`** hoặc kế thừa Class **`Thread`**.

```java
import java.net.*;
import java.io.*;

public class MultiThreadedServer {
    public static void main(String[] args) throws IOException {
        try (ServerSocket serverSocket = new ServerSocket(6666)) {
            System.out.println("Server đang lắng nghe tại port 6666...");
            while (true) {
                // Main Thread liên tục lắng nghe
                Socket clientSocket = serverSocket.accept(); 
                System.out.println("Client mới kết nối: " + clientSocket.getRemoteSocketAddress());
                
                // Tạo Thread mới cho mỗi Client
                new Thread(new ClientHandler(clientSocket)).start(); 
            }
        } catch (IOException e) {
            System.err.println("Lỗi Server: " + e.getMessage());
        }
    }
}

class ClientHandler implements Runnable {
    private Socket clientSocket;

    public ClientHandler(Socket socket) {
        this.clientSocket = socket;
    }

    @Override
    public void run() {
        // Logíc xử lý giao tiếp I/O với Client đó ở đây
        try (BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
             PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true)) {
             
            String message;
            while ((message = in.readLine()) != null) {
                System.out.println("Nhận từ Client " + clientSocket.getRemoteSocketAddress() + ": " + message);
                out.println("Đã nhận: " + message);
                if ("quit".equalsIgnoreCase(message)) break;
            }
        } catch (IOException e) {
            System.out.println("Lỗi xử lý Client: " + e.getMessage());
        } finally {
            try { clientSocket.close(); } catch (IOException e) {}
            System.out.println("Client " + clientSocket.getRemoteSocketAddress() + " ngắt kết nối.");
        }
    }
}
```

## Các từ khóa cần nhớ

- **Multi-Threading**: Khả năng thực thi nhiều phần của chương trình đồng thời.
- **Blocking I/O**: Hoạt động đọc/ghi dữ liệu tạm dừng luồng cho đến khi hoàn thành.
- **Runnable/Thread**: Các Class/Interface cơ bản để tạo luồng trong Java.
- **Client Handler**: Một luồng riêng biệt chịu trách nhiệm xử lý giao tiếp với một Client cụ thể.

## Vận dụng

- Xây dựng bất kỳ Server thực tế nào: Web Server, Mail Server, Game Server.
- Tối ưu hóa hiệu suất ứng dụng mạng bằng cách phân tán tải xử lý.

## Tóm tắt kiến thức

Trong lập trình Socket Java, **Multi-Threading** là bắt buộc để Server có thể xử lý nhiều Client đồng thời. Bằng cách gán một **Thread** riêng cho mỗi Client mới, ta tránh được hiện tượng **Blocking I/O** làm tắc nghẽn toàn bộ Server.

## Nguồn tham khảo

- [tutorialspoint Documentation: Java - Multithreading](https://www.tutorialspoint.com/java/java_multithreading.htm)
- [Multithreading trong ngôn ngữ java](https://viblo.asia/p/multithreading-trong-ngon-ngu-java-157G5oz3RAje)