---
title1: "JS JSON" 
title: "JavaScript và JSON: Cẩm Nang Làm Việc Với 'Ngôn Ngữ' Của API"
description: "Tìm hiểu JSON là gì, và cách sử dụng hai phương thức cốt lõi JSON.stringify() và JSON.parse() để tuần tự hóa và giải tuần tự hóa dữ liệu trong JavaScript."
date: 2025-10-31
tags: ["JavaScript", "JSON", "stringify", "parse", "API", "Serialization"]
cover: "/images/blog/javascript-json.png"
draft: false
---

Trong thế giới web, các ứng dụng cần trao đổi dữ liệu. Làm thế nào để một Server viết bằng Python gửi thông tin cho một trình duyệt chạy JavaScript? Họ cần một ngôn ngữ chung, và **JSON (JavaScript Object Notation)** chính là ngôn ngữ đó. Mặc dù có tên là "JavaScript", JSON thực chất là một định dạng văn bản (text format) độc lập, siêu nhẹ và dễ đọc.

## Mục tiêu bài học

- Hiểu rõ JSON là gì và quy tắc cú pháp của nó.
- Nắm vững hai phương thức cốt lõi: **`JSON.stringify()`** và **`JSON.parse()`**.
- Biết cách xử lý lỗi khi làm việc với JSON.

## 1. JSON Là Gì?

JSON là một định dạng văn bản dùng để lưu trữ và truyền tải dữ liệu. Cú pháp của nó dựa trên cách JavaScript tạo đối tượng, nhưng **nghiêm ngặt hơn**:
* Chỉ bao gồm **Cặp Key-Value** (ví dụ: `"name": "John"`) hoặc **Mảng** (ví dụ: `[1, "hello", null]`).
* **Key (Khóa)** phải luôn nằm trong dấu ngoặc kép (`""`).
* **Value (Giá trị)** có thể là: chuỗi (`""`), số, `true`, `false`, `null`, mảng, hoặc một đối tượng JSON khác.
* **Không** cho phép hàm (functions), `undefined`, comment, hay dấu phẩy thừa (trailing comma).

```json
{
  "id": 1,
  "name": "Leanne Graham",
  "isActive": true,
  "skills": ["JavaScript", "HTML", "CSS"],
  "address": {
    "city": "Hanoi"
  },
  "manager": null
}
```

## 2. `JSON.stringify()`: Từ JavaScript Object sang JSON String

Khi bạn muốn gửi dữ liệu (ví dụ: một đối tượng JavaScript) đến Server, bạn phải chuyển nó thành một chuỗi JSON. Chúng ta sử dụng `JSON.stringify()`.

**Serialization (Tuần tự hóa)**: Quá trình chuyển đổi một đối tượng trong bộ nhớ (như JS Object) thành một định dạng (như chuỗi JSON) để có thể lưu trữ hoặc truyền đi.

```javascript
const user = {
    name: "Alice",
    age: 30,
    isAdmin: false,
    sayHello: function() { console.log('Hi'); }, // Sẽ bị bỏ qua
    lastLogin: undefined // Sẽ bị bỏ qua
};

// Chuyển đối tượng JS thành chuỗi JSON
const jsonString = JSON.stringify(user);

console.log(jsonString);
// Output: {"name":"Alice","age":30,"isAdmin":false}
```

## 3. JSON.parse(): Từ JSON String sang JavaScript Object

Khi bạn nhận dữ liệu từ một API (thường là một chuỗi JSON), bạn cần chuyển nó trở lại thành một đối tượng JavaScript để có thể sử dụng. Chúng ta dùng `JSON.parse()`.

**Deserialization (Giải tuần tự hóa)**: Quá trình ngược lại, chuyển đổi chuỗi JSON trở lại thành đối tượng JavaScript.

```javascript
const jsonFromApi = '{"id": 1, "name": "Bob", "hobbies": ["Reading", "Gaming"]}';

// Chuyển chuỗi JSON thành đối tượng JS
const userObject = JSON.parse(jsonFromApi);

console.log(userObject.name); // Output: Bob
console.log(userObject.hobbies[0]); // Output: Reading
```

Xử lý lỗi: Nếu chuỗi JSON không hợp lệ (ví dụ: thiếu dấu ngoặc kép ở key), `JSON.parse()` sẽ ném ra lỗi **SyntaxError**. Luôn sử dụng `try...catch` khi parse dữ liệu không đáng tin cậy.

```javascript
const badJson = '{ "name": "Eve", age: 25 }'; // Lỗi: key 'age' thiếu ""

try {
    const user = JSON.parse(badJson);
} catch (error) {
    console.error("Lỗi parse JSON:", error.message); 
    // Output: Lỗi parse JSON: Unexpected token a in JSON at position 18
}
```

## Các từ khóa cần nhớ

- **JSON: (JavaScript Object Notation)** Định dạng trao đổi dữ liệu văn bản.
- **`JSON.stringify()`**: (Serialization) Chuyển Object/Array của JS thành chuỗi JSON.
- **`JSON.parse()`**: (Deserialization) Chuyển chuỗi JSON hợp lệ thành Object/Array của JS.
- **Serialization**: Quá trình chuyển đổi đối tượng thành chuỗi/dạng lưu trữ.
- **`try...catch`**: Dùng để bắt lỗi khi parse JSON.

## Vận dụng

- Gửi và nhận dữ liệu từ các API (ví dụ: dùng `fetch()`).
- Lưu trữ dữ liệu phức tạp trong **localStorage** (vì **localStorage** chỉ lưu được chuỗi).
- Đọc các file cấu hình (ví dụ: `package.json`).

## Tóm tắt kiến thức

- **JSON** là định dạng dữ liệu chuẩn của web. 
- JavaScript cung cấp hai phương thức chính để làm việc với nó: `JSON.stringify()` để "đóng gói" đối tượng JS thành chuỗi, và `JSON.parse()` để **"mở gói"** chuỗi JSON trở lại thành đối tượng JS.

## Nguồn tham khảo
- [JSON Object](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Scripting/JSON)
- [json.org](https://www.json.org/json-en.html)
