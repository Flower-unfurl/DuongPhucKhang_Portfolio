---
title1:"Java-network" 
title: "Non-Blocking I/O (NIO) Trong Java: Tối Ưu Hóa Hiệu Năng Server"
description: "Tìm hiểu Java NIO (Non-Blocking I/O) và cách nó vượt trội hơn mô hình Blocking I/O truyền thống bằng cách sử dụng Channel, Buffer, và Selector."
date: 2025-10-25
tags: ["Java", "Network", "NIO", "Non-Blocking", "Selector", "Channel", "Buffer"]
cover: "/images/blog/java-network-nio.png"
draft: false
---

Với sự phát triển của các hệ thống quy mô lớn, mô hình **Blocking I/O** truyền thống (sử dụng Thread cho mỗi Client) bắt đầu lộ ra nhược điểm: chi phí tạo và quản lý hàng nghìn Thread là rất lớn. Java đã giới thiệu **Non-Blocking I/O (NIO)**, cho phép một số lượng nhỏ các Thread xử lý một lượng lớn các kết nối đồng thời.

## Mục tiêu bài học

- Hiểu vấn đề về hiệu suất của mô hình I/O truyền thống (Blocking I/O).
- Giới thiệu các thành phần cốt lõi của Java NIO: **Channel, Buffer, Selector**.
- Nắm được nguyên lý hoạt động của **Selector** để quản lý nhiều kết nối.

## 1. Blocking I/O vs. Non-Blocking I/O (NIO)

| Tính chất | Blocking I/O (I/O Truyền thống) | Non-Blocking I/O (NIO) |
| :--- | :--- | :--- |
| **I/O Call** | Hàm gọi bị **treo (block)** cho đến khi I/O hoàn thành. | Hàm gọi **trả về ngay lập tức**, có thể chưa hoàn thành. |
| **Kết nối** | Một Thread cho mỗi kết nối (tốn tài nguyên). | Một số ít Thread xử lý **nhiều kết nối** (hiệu quả hơn). |
| **Sử dụng** | **`InputStream`/`OutputStream`**, **`Socket`** | **`Channel`, `Buffer`, `Selector`** |

## 2. Các Thành Phần Chính của Java NIO

1. **Channels (Kênh)**: Là "kênh" hai chiều để giao tiếp với các thực thể I/O (như file, socket). Thay thế cho Stream. Ví dụ: `SocketChannel`, `ServerSocketChannel`.
2. **Buffers (Bộ đệm)**: Là vùng nhớ nơi dữ liệu được đọc từ Channel hoặc ghi vào Channel. Dữ liệu phải được đặt vào Buffer trước khi có thể gửi đi, và được đọc từ Buffer sau khi nhận về.
3. **Selectors (Bộ chọn)**: Đây là trái tim của NIO Server. Một `Selector` có thể giám sát **nhiều `Channel`** (kết nối) và cho biết Channel nào đã sẵn sàng cho một hoạt động I/O cụ thể (ví dụ: sẵn sàng đọc, sẵn sàng ghi, sẵn sàng chấp nhận kết nối).

## 3. Nguyên Lý Hoạt Động Của Selector

Với NIO, Server chỉ cần một hoặc một vài Thread chạy vòng lặp vô tận, gọi `Selector.select()`.

1. **Đăng ký (Register)**: Các `Channel` (kết nối) được đăng ký với `Selector` để theo dõi các sự kiện quan tâm (`OP_ACCEPT`, `OP_READ`, `OP_WRITE`).
2. **Chọn (Select)**: `select()` sẽ **chờ** (blocking, nhưng chỉ chờ cho đến khi bất kỳ Channel nào sẵn sàng) và trả về tập hợp các Channel đang sẵn sàng.
3. **Xử lý (Process)**: Server xử lý các sự kiện I/O trên các Channel sẵn sàng, và sau đó quay lại bước `select()`.

## 4. Code Snippet Cơ Bản (ServerSocketChannel với Selector)

```java
import java.net.InetSocketAddress;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.util.Iterator;
import java.util.Set;

public class NIOServerExample {
    public static void main(String[] args) throws Exception {
        // Khởi tạo ServerSocketChannel và Selector
        ServerSocketChannel serverChannel = ServerSocketChannel.open();
        serverChannel.configureBlocking(false); // Rất quan trọng: Set Non-blocking
        serverChannel.bind(new InetSocketAddress(8888));

        Selector selector = Selector.open();
        // Đăng ký sự kiện chấp nhận kết nối
        serverChannel.register(selector, SelectionKey.OP_ACCEPT); 
        System.out.println("NIO Server đang lắng nghe tại port 8888...");

        while (true) {
            selector.select(); // Chờ Channel sẵn sàng
            Set<SelectionKey> selectedKeys = selector.selectedKeys();
            Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

            while (keyIterator.hasNext()) {
                SelectionKey key = keyIterator.next();
                
                if (key.isAcceptable()) {
                    // Xử lý chấp nhận kết nối (tạo SocketChannel)
                    // ...
                } else if (key.isReadable()) {
                    // Xử lý đọc dữ liệu từ SocketChannel
                    // ...
                }
                keyIterator.remove();
            }
        }
    }
}
```

## Các từ khóa cần nhớ

- **NIO**: Non-Blocking I/O, mô hình hiệu suất cao, không dùng Thread cho mỗi kết nối.
- **Channel**: Kênh giao tiếp hai chiều (thay thế Stream).
- **Buffer**: Bộ đệm chứa dữ liệu (phải dùng Buffer để đọc/ghi).
- **Selector**: Công cụ quản lý, giám sát trạng thái I/O của nhiều Channel.
- **OP_ACCEPT, OP_READ, OP_WRITE**: Các loại sự kiện I/O.

## Vận dụng

- Xây dựng các Server hiệu suất cao, có khả năng xử lý hàng chục nghìn kết nối đồng thời (ví dụ: Netty, Apache MINA được xây dựng trên NIO).

## Tóm tắt kiến thức

Java **NIO** là một bước tiến lớn trong lập trình mạng, giúp tối ưu hóa hiệu suất Server bằng cách thay thế mô hình **Thread-per-connection** bằng kiến trúc `Selector` và `Channel`, cho phép một số lượng nhỏ các Thread quản lý lượng lớn I/O không chặn.

## Nguồn tham khảo

- [Oracle Java NIO](https://docs.oracle.com/javase/tutorial/essential/io/fileio.html)
- [Java.nio package Documentation](https://docs.oracle.com/javase/8/docs/api/java/nio/package-summary.html)
