---
title1: "JS Async/Await" 
title: "Promises, Async/Await: Làm Chủ Lập Trình Bất Đồng Bộ Hiện Đại"
description: "Thoát khỏi Callback Hell với Promises và làm chủ cú pháp `async/await` trong JavaScript. Đây là mô hình chuẩn mực để xử lý các tác vụ bất đồng bộ."
date: 2025-10-29
tags: ["JavaScript", "Async", "Await", "Promise", "ES2017", "Callback Hell"]
cover: "/images/blog/javascript-async-await.png"
draft: false
---

Nếu bạn đã từng đau đầu với **Callback Hell** (lồng ghép callback liên tiếp), bạn sẽ đánh giá cao cách **Promises** và cú pháp **`async/await`** cách mạng hóa lập trình bất đồng bộ trong JavaScript. Đây là mô hình chuẩn mực để xử lý các tác vụ như gọi API, đọc file, hay `setTimeout`.

## Mục tiêu bài học

- Hiểu mục đích của **Promise** và các trạng thái của nó.
- Chuyển đổi từ mô hình **Callback** sang **Promise**.
- Sử dụng cú pháp **`async/await`** để viết code bất đồng bộ như đồng bộ.

## 1. Promises: Lời Hứa Cho Kết Quả Tương Lai

**Promise** là một đối tượng đại diện cho việc hoàn thành (hoặc thất bại) của một thao tác bất đồng bộ và giá trị kết quả của nó.

Một Promise có 3 trạng thái:
1. **Pending (Đang chờ)**: Thao tác bất đồng bộ chưa hoàn thành.
2. **Fulfilled (Đã hoàn thành)**: Thao tác thành công, Promise trả về một giá trị. (Gọi `.then(value => ...)`).
3. **Rejected (Bị từ chối)**: Thao tác thất bại, Promise trả về một lý do (Error). (Gọi `.catch(error => ...)`).

Sử dụng Promises giúp chúng ta thoát khỏi Callback Hell bằng cách cho phép **xâu chuỗi các thao tác bất đồng bộ** một cách tuần tự (Promise Chaining).

```javascript
// Ví dụ Promise Chaining
fetch('[https://api.example.com/data](https://api.example.com/data)') // fetch trả về một Promise
  .then(response => response.json()) // Trả về Promise mới
  .then(data => {
    console.log('Dữ liệu đã sẵn sàng:', data);
  })
  .catch(error => { // Xử lý lỗi cho toàn bộ chuỗi
    console.error('Có lỗi xảy ra:', error);
  });
  ```

## 2. Async/Await: Bất Đồng Bộ Cực Kỳ Dễ Đọc

- `async/await` được giới thiệu trong ES2017. Nó là **"Syntactic Sugar"** (lớp đường cú pháp) cho Promises, cho phép bạn viết code bất đồng bộ gần giống như code đồng bộ.
- `async`: Đặt trước một function để biến nó thành một **Async Function**. Một Async Function luôn trả về một Promise.
- `await`: Chỉ có thể được sử dụng bên trong một Async Function. Nó tạm dừng việc thực thi của Async Function cho đến khi Promise được đặt trước nó hoàn thành (fulfilled hoặc rejected).

```javascript
async function fetchData() {
    try {
        // await tạm dừng tại đây cho đến khi Promise của fetch hoàn thành
        const response = await fetch('[https://api.example.com/data](https://api.example.com/data)'); 
        
        // await tiếp tục tạm dừng cho đến khi response.json() hoàn thành
        const data = await response.json(); 
        
        console.log('Dữ liệu đã sẵn sàng:', data);
        return data; // Async function tự động bọc giá trị trả về
    } catch (error) {
        // Dùng try...catch để bắt lỗi (Rejected Promise)
        console.error('Đã xảy ra lỗi:', error);
        throw error;
    }
}

// Gọi hàm async:
fetchData().then(data => console.log('Thành công!'));
```

## 3. Các Phương Thức Hỗ Trợ Khác

- `Promise.all([p1, p2, p3])`: Chờ tất cả các Promises hoàn thành. Nếu một Promise bị từ chối, toàn bộ `Promise.all` sẽ bị từ chối ngay lập tức.
- `Promise.race([p1, p2, p3])`: Trả về kết quả của Promise hoàn thành đầu tiên (dù fulfilled hay rejected).

## Các từ khóa cần nhớ

- **Promise**: Đối tượng đại diện cho kết quả bất đồng bộ. Trạng thái: Pending, Fulfilled, Rejected.
- **Promise Chaining**: Sử dụng `.then()` để xâu chuỗi các thao tác bất đồng bộ.
- **Callback Hell**: Hiện tượng lồng ghép quá nhiều callback gây khó đọc code.
- `async`: Từ khóa biến hàm thành Async Function (luôn trả về Promise).
- `await`: Tạm dừng Async Function cho đến khi Promise hoàn thành.

## Vận dụng

- Thay thế mọi logic Callback Hell bằng Promises hoặc `async/await`.
- Sử dụng `Promise.all` để tối ưu hóa hiệu suất bằng cách thực thi nhiều API calls độc lập đồng thời.

## Tóm tắt kiến thức

- **Promises** cung cấp một giải pháp cấu trúc hơn cho lập trình bất đồng bộ so với callbacks. 
- `async/await` xây dựng trên Promises, cho phép lập trình viên viết code bất đồng bộ với cú pháp gần gũi với code đồng bộ, cải thiện đáng kể khả năng đọc và bảo trì mã nguồn.

## Nguồn tham khảo
- [MDN Web Docs: Using Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)
- [JavaScript.info: Async/await](https://javascript.info/async-await)




