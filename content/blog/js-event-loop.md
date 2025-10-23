---
title1: "JS Event Loop" 
title: "JavaScript: Khám Phá Event Loop - Động Cơ Vận Hành Bất Đồng Bộ"
description: "Hiểu rõ cách JavaScript, dù là đơn luồng, vẫn xử lý được các tác vụ bất đồng bộ mà không làm treo UI, thông qua cơ chế Event Loop, Call Stack, và Callback Queue."
date: 2025-10-26
tags: ["JavaScript", "Event Loop", "Async", "Callback Queue", "Call Stack", "Microtask"]
cover: "/images/blog/javascript-eventloop.png"
draft: false
---

JavaScript nổi tiếng là ngôn ngữ **đơn luồng (Single-Threaded)**. Vậy làm thế nào nó có thể xử lý các tác vụ tốn thời gian như gọi API (I/O) mà không làm giao diện người dùng **bị treo (freeze)**? Câu trả lời nằm ở cơ chế kỳ diệu mang tên **Event Loop**. Hiểu rõ Event Loop là chìa khóa để làm chủ lập trình bất đồng bộ (Asynchronous) trong JS.

## Mục tiêu bài học

- Hiểu **JavaScript Runtime** bao gồm những gì.
- Nắm được vai trò của **Call Stack, Web APIs, Callback Queue,** và **Event Loop**.
- Phân tích thứ tự thực thi của các hàm bất đồng bộ.

## 1. Cấu Trúc JavaScript Runtime

Để hiểu Event Loop, ta cần biết các thành phần cơ bản của môi trường thực thi JavaScript (Runtime - ví dụ: trình duyệt hoặc Node.js):

- **Call Stack (Ngăn xếp gọi)**: Nơi các hàm được gọi sẽ được xếp vào hàng đợi và thực thi theo nguyên tắc LIFO (Last-In, First-Out). Vì JS là đơn luồng, Call Stack chỉ có một.
- **Web APIs/Node APIs**: Các môi trường bên ngoài cung cấp các hàm bất đồng bộ như `setTimeout()`, DOM events, `fetch()`. Các tác vụ nặng sẽ được chuyển cho API này xử lý.
- **Callback Queue (Hàng đợi Callback)**: Nơi các hàm callback đã hoàn thành của Web APIs được xếp hàng để chờ Call Stack rảnh.
- **Event Loop (Vòng lặp Sự kiện)**: Cơ chế liên tục kiểm tra xem Call Stack có **rỗng** không. Nếu rỗng, Event Loop sẽ lấy hàm đầu tiên từ Callback Queue và đẩy nó vào Call Stack để thực thi.

## 2. Mô Phỏng Quy Trình Event Loop

Hãy xem đoạn mã sau:

```javascript
console.log('A');

setTimeout(function callback() {
  console.log('B');
}, 0); // Đặt timeout là 0ms

console.log('C');

// Output sẽ là: A, C, B
```

**Phân tích thứ tự**:
1. **Call Stack**: `console.log('A')` được thực thi và in ra A.
2. **Call Stack**: `setTimeout(callback, 0)` được gọi. Hàm setTimeout được chuyển cho Web APIs. Web APIs bắt đầu đếm ngược 0ms (thực tế là tối thiểu).
3. **Call Stack**: `console.log('C')` được thực thi và in ra C. Call Stack trở nên rỗng.
4. **Web APIs**: Sau 0ms, hàm `callback()` được chuyển sang Callback Queue.
5. **Event Loop**: Phát hiện Call Stack **rỗng** và Callback Queue **có nội dung**.
6. **Event Loop**: Đẩy `callback()` từ Queue vào Call Stack.
7. **Call Stack**: `console.log('B')` được thực thi và in ra B.

## 3. Microtask Queue và Macrotask Queue (Nâng cao)
Thực tế có hai loại Queue:
- **Macrotask Queue (Callback Queue)**: Chứa các callback từ `setTimeout()`, `setInterval()`, DOM events.
- **Microtask Queue**: Chứa các callback từ Promises (`.then()`, `.catch()`, `.finally()`) và `queueMicrotask()`.
**Quy tắc ưu tiên**: Event Loop sẽ luôn luôn ưu tiên chạy hết tất cả các tác vụ trong **Microtask Queue** trước, sau đó mới đến lượt một tác vụ duy nhất từ **Macrotask Queue**.

## Các từ khóa cần nhớ

- **Single-Threaded**: JavaScript chỉ có một luồng thực thi chính (Call Stack).
- **Call Stack**: Nơi các hàm đồng bộ được thực thi.
- **Web APIs**: Môi trường xử lý các tác vụ bất đồng bộ (setTimeout, fetch).
- **Event Loop**: Cơ chế kiểm tra Call Stack rỗng và đẩy callback từ Queue vào Stack.
- **Microtask Queue**: Ưu tiên cao hơn Macrotask Queue (chủ yếu là Promises).

## Vận dụng

- Phát triển các ứng dụng có phản hồi nhanh, không bị treo (non-blocking UI).
- Hiểu rõ thứ tự thực thi khi làm việc với Promises, **async/await** (chính là syntactic sugar cho Promises) và **setTimeout**.

## Nguồn tham khảo

- [Philip Roberts: "What the heck is the event loop anyway?](https://www.youtube.com/watch?v=8aGhZQkoFbQ)
- [MDN Web Docs: Concurrency model and Event Loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Execution_model)