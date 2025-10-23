---
title1: "JS Scope and more" 
title: "Scope, Closure và Hoisting: Ba Khái Niệm Quan Trọng Nhất Của JavaScript"
description: "Làm chủ ba khái niệm nền tảng của JavaScript: Scope (Global, Function, Block), Closure (Bao đóng) và Hoisting (Kéo lên), và hiểu rõ sự khác biệt giữa var, let, và const."
date: 2025-10-28
tags: ["JavaScript", "Scope", "Closure", "Hoisting", "let", "var", "const"]
cover: "/images/blog/javascript-closure-scope-hoisting.png"
draft: false
---

Bạn có biết tại sao một biến đôi khi có thể được truy cập từ bên ngoài hàm, hoặc tại sao bạn có thể gọi một hàm trước khi định nghĩa nó? Ba khái niệm **Scope, Closure, và Hoisting** là câu trả lời. Chúng định hình cách JS quản lý và truy cập các biến, và là nền tảng của các thiết kế mẫu (Design Pattern) nâng cao.

## Mục tiêu bài học

- Hiểu các loại **Scope** khác nhau trong JS (**Global, Function, Block**).
- Nắm được cách **Hoisting** hoạt động với `var`, `let`, `const`.
- Làm chủ **Closure** và ứng dụng thực tế của nó.

## 1. Scope (Phạm vi)

Scope xác định **khả năng truy cập** của biến và hàm tại các phần khác nhau của code.

- **Global Scope**: Biến được khai báo ngoài bất kỳ hàm nào. Có thể truy cập từ mọi nơi.
- **Function Scope (Phạm vi Hàm)**: Các biến được khai báo bên trong một hàm (với từ khóa `var`) chỉ có thể được truy cập bên trong hàm đó.
- **Block Scope (Phạm vi Khối)**: Được tạo ra bởi các khối `{}` (ví dụ: `if`, `for`, `while`). Các từ khóa **`let`** và **`const`** là **Block-Scoped**, nghĩa là chúng chỉ tồn tại bên trong khối mà chúng được khai báo.

```javascript
var globalVar = 'Toàn cục';

function myFunction() {
    var functionVar = 'Phạm vi hàm';
    
    if (true) {
        let blockLet = 'Phạm vi khối';
        console.log(functionVar); // OK
    }
    // console.log(blockLet); // LỖI! let chỉ trong khối if {}
}
```

## 2. Hoisting (Kéo lên)

Hoisting là cơ chế mặc định của JavaScript, di chuyển các khai báo lên đầu phạm vi hiện tại trước khi code được thực thi.
- **Hàm**: Toàn bộ hàm được hoist (khai báo và định nghĩa).
- **var**: Chỉ **khai báo biến** được hoist. Việc **gán giá trị** vẫn ở vị trí cũ.
- **let/const**: Khai báo cũng được hoist nhưng không được khởi tạo (initialize). Chúng nằm trong **Temporal Dead Zone (TDZ)** cho đến khi gặp dòng khai báo.

```javascript
// Ví dụ Hoisting
console.log(a); // Output: undefined (Chỉ khai báo `var a;` được hoist)
var a = 10;

// console.log(b); // LỖI ReferenceError (b ở trong TDZ)
let b = 20;

myFunc(); // OK, toàn bộ function được hoist
function myFunc() {
    console.log("Hàm được gọi trước khi định nghĩa");
}
```

## 3. Closure (Bao đóng)

**Closure** là khả năng của một hàm **ghi nhớ và truy cập** các biến từ **phạm vi bên ngoài (lexical environment)** nơi nó được định nghĩa, ngay cả sau khi hàm bên ngoài đã thực thi xong.

```javascript
function outerFunction(outerVar) {
    // outerFunction đã thực thi xong, nhưng innerFunction vẫn giữ tham chiếu đến outerVar
    return function innerFunction(innerVar) {
        console.log(`Bên ngoài: ${outerVar}, Bên trong: ${innerVar}`);
    }
}

const closureExample = outerFunction('Xin chào');
closureExample('Thế giới'); // Output: Bên ngoài: Xin chào, Bên trong: Thế giới
```
Closure cho phép chúng ta tạo ra các Hàm factory hoặc mô phỏng Private Methods (phương thức riêng tư).

## Các từ khóa cần nhớ
- **Scope**: Quy tắc truy cập biến (Global, Function, Block).
- **let/const**: Giới thiệu Block Scope.
- **Hoisting**: Cơ chế di chuyển khai báo lên đầu scope.
- **TDZ**: Temporal Dead Zone, khu vực cấm truy cập let/const trước khi khai báo.
- **Closure**: Hàm ghi nhớ và truy cập các biến từ phạm vi bên ngoài nó.

## Vận dụng

- Sử dụng **Closure** để tạo ra các hàm đếm (counters) hay module với dữ liệu riêng tư (Private data).
- Luôn dùng **let/const** thay vì **var** để tránh các lỗi logic do Function Scope và Hoisting không mong muốn.

## Tóm tắt kiến thức

- **Scope** quản lý khả năng truy cập biến. 
- **Hoisting** định hình cách khai báo biến được xử lý.  
- **Closure** tận dụng **Scope** để tạo ra các hàm mạnh mẽ có thể duy trì trạng thái riêng tư, là cốt lõi của lập trình hàm (Functional Programming) trong JS.

## Nguồn tham khảo

- [MDN Web Docs: Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures)
- [Understanding let, const, and var in JavaScript](https://www.freecodecamp.org/news/var-let-and-const-whats-the-difference/)
