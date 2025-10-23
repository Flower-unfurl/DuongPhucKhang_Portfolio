---
title1:"JavaScript" 
title: "Lập Trình Hàm (FP) Trong JavaScript: Sức Mạnh của Immutability và Higher-Order Functions"
description: "Khám phá Lập trình hàm (Functional Programming) trong JavaScript. Tìm hiểu về Pure Functions, Immutability, và các Higher-Order Functions như map, filter, reduce."
date: 2025-10-30
tags: ["JavaScript", "Functional Programming", "map", "filter", "reduce", "Immutability", "Pure Functions"]
cover: "/images/blog/javascript-functional.png"
draft: false
---

JavaScript không chỉ là một ngôn ngữ hướng đối tượng mà còn hỗ trợ mạnh mẽ **Lập trình hàm (FP - Functional Programming)**. FP nhấn mạnh vào việc sử dụng các hàm như công dân hạng nhất, tránh thay đổi trạng thái (Mutability), và xây dựng logic bằng cách kết hợp các hàm. Đây là phong cách lập trình đang thống trị các framework hiện đại như React.

## Mục tiêu bài học

- Hiểu khái niệm **First-Class Functions** (Hàm là công dân hạng nhất).
- Làm quen với **Immutability** (Bất biến) và **Pure Functions** (Hàm thuần khiết).
- Áp dụng các **Higher-Order Functions** cơ bản (`map`, `filter`, `reduce`).

## 1. Hàm là Công Dân Hạng Nhất (First-Class Functions)

Trong JavaScript, hàm là **First-Class Citizens**, nghĩa là bạn có thể:
1. Gán hàm cho một biến.
2. Truyền hàm như một đối số cho hàm khác.
3. Trả về hàm từ một hàm khác.

Khả năng này cho phép các **Higher-Order Functions** (Hàm bậc cao) ra đời.

## 2. Higher-Order Functions (HOFs)

HOFs là các hàm nhận một hoặc nhiều hàm khác làm đối số, hoặc trả về một hàm. Đây là công cụ cơ bản nhất trong FP.

- **`map()`**: Dùng để **biến đổi** (Transform) các phần tử của một mảng và trả về một mảng mới có cùng độ dài.
    ```javascript
    const numbers = [1, 2, 3];
    const squares = numbers.map(num => num * num); // [1, 4, 9]
    ```

- **`filter()`**: Dùng để **lọc** (Filter) các phần tử theo điều kiện và trả về một mảng con.
    ```javascript
    const ages = [18, 15, 22, 16];
    const adults = ages.filter(age => age >= 18); // [18, 22]
    ```

- **`reduce()`**: Dùng để **kết hợp** (Combine/Reduce) tất cả các phần tử của một mảng thành một giá trị duy nhất.
    ```javascript
    const sum = numbers.reduce((accumulator, num) => accumulator + num, 0); // 6
    ```

## 3. Pure Functions và Immutability

FP hướng đến việc viết **Pure Functions** và duy trì **Immutability**.

- **Pure Functions (Hàm thuần khiết)**:
    1. Với cùng một đầu vào, luôn trả về cùng một đầu ra (Không phụ thuộc vào trạng thái bên ngoài).
    2. Không gây ra **side effects** (tác dụng phụ) như thay đổi biến toàn cục, ghi vào console, gọi API, hay sửa đổi đối số.

- **Immutability (Tính bất biến)**: Tránh thay đổi dữ liệu gốc. Khi cần thay đổi, hãy tạo một **bản sao mới** của dữ liệu.
    ```javascript
    const originalArray = [1, 2, 3];
    
    // BAD (Thay đổi mảng gốc)
    originalArray.push(4); 
    
    // GOOD (Sử dụng Immutability - Tạo mảng mới)
    const newArray = [...originalArray, 4]; // [1, 2, 3, 4]
    ```

## 4. Function Composition (Kết hợp Hàm)

FP khuyến khích xây dựng logic bằng cách kết hợp (chaining) các hàm thuần khiết.

```javascript
const transactions = [100, -50, 200, 30];

// Lọc các giao dịch dương VÀ nhân đôi giá trị
const result = transactions
  .filter(t => t > 0)
  .map(t => t * 2)
  .reduce((acc, t) => acc + t, 0); 
  
console.log(result); // (100*2) + (200*2) + (30*2) = 660
```

## Các từ khóa cần nhớ
- **Functional Programming (FP)**: Phong cách lập trình tập trung vào việc sử dụng các hàm.
- **First-Class Functions**: Hàm có thể được gán, truyền, và trả về như mọi giá trị khác.
- **Higher-Order Functions**: Hàm nhận hoặc trả về hàm (ví dụ: `map`, `filter`).
- **Pure Functions**: Hàm không gây side effect và trả về cùng kết quả với cùng input.
- **Immutability**: Tránh thay đổi dữ liệu gốc, thay vào đó tạo bản sao mới.

## Vận dụng

- Viết code React/Redux với các hàm reducer thuần khiết.
- Xây dựng các chuỗi xử lý dữ liệu (data pipelines) dễ đọc, dễ kiểm thử.

## Tóm tắt kiến thức

- Lập trình hàm (FP) trong JavaScript tận dụng **First-Class Functions** để sử dụng Higher-Order Functions như `map`, `filter`, `reduce`. 
- FP đề cao việc sử dụng **Pure Functions** và giữ cho dữ liệu **bất biến (Immutable)**, giúp code trở nên mạnh mẽ, dễ dự đoán và dễ mở rộng.

## Nguồn tham khảo

- [Array.prototype.map()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)
- [Functional Programming trong Javascript](https://viblo.asia/p/functional-programming-trong-javascript-YWOZrB9vZQ0)
