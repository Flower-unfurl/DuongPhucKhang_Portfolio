---
title1: "JS Prototype" 
title: "JavaScript: Khai Thác Sức Mạnh của Prototype và Kế Thừa"
description: "Hiểu rõ mô hình Kế thừa dựa trên Prototype (Prototypal Inheritance) của JavaScript. Phân biệt `prototype` và `__proto__`, và cách ES6 Class chỉ là cú pháp."
date: 2025-10-27
tags: ["JavaScript", "Prototype", "Inheritance", "ES6 Class", "__proto__", "OOP"]
cover: "/images/blog/javascript-prototype.png"
draft: false
---

Nếu bạn đến từ một ngôn ngữ hướng đối tượng (OOP) truyền thống như Java, bạn sẽ thấy mô hình kế thừa của JavaScript có chút khác biệt. JS sử dụng mô hình **Kế thừa dựa trên Prototype (Prototypal Inheritance)** thay vì kế thừa dựa trên Class. Hiểu được `prototype` là chìa khóa để nắm vững cách các đối tượng và tính năng OOP hoạt động trong JS.

## Mục tiêu bài học

- Phân biệt **Kế thừa dựa trên Class** và **Kế thừa dựa trên Prototype**.
- Hiểu thuộc tính **`__proto__`** và **`prototype`**.
- Sử dụng **Classes (ES6)** như một lớp "đường" cú pháp (Syntactic Sugar).

## 1. Kế Thừa Dựa Trên Prototype là gì?

Trong JS, khi bạn cố gắng truy cập một thuộc tính hoặc phương thức của một đối tượng:
1. JS kiểm tra xem thuộc tính/phương thức đó có tồn tại trên **đối tượng hiện tại** không.
2. Nếu không tìm thấy, JS sẽ di chuyển lên đối tượng được tham chiếu bởi thuộc tính **`__proto__`** (hay còn gọi là **Prototype Link**) của đối tượng hiện tại.
3. Quá trình này tiếp tục cho đến khi tìm thấy thuộc tính/phương thức hoặc đạt đến cuối chuỗi prototype (`null`). Chuỗi các đối tượng này được gọi là **Prototype Chain (Chuỗi Prototype)**.

## 2. `prototype` vs. `__proto__`

Đây là hai thuộc tính gây nhầm lẫn nhất:

- **`prototype` (Thuộc tính của Constructor/Function)**: Thuộc tính này chỉ tồn tại trên **hàm tạo (Constructor Function)**. Bất kỳ đối tượng nào được tạo ra từ hàm tạo này sẽ **liên kết** (qua `__proto__`) đến đối tượng được tham chiếu bởi `Constructor.prototype`.
- **`__proto__` (Thuộc tính của Object/Instance)**: Thuộc tính này là liên kết thực tế đến **prototype của nó**. Nó trỏ đến đối tượng mà nó kế thừa các thuộc tính và phương thức.

```javascript
function Animal(name) { // Constructor Function
    this.name = name;
}

Animal.prototype.sound = function() { // Thêm method vào prototype
    console.log("Animal makes a sound");
};

const dog = new Animal("Dog"); // dog là instance

// 1. dog có thuộc tính name riêng
console.log(dog.name); // Output: Dog

// 2. dog không có method sound, nên nó đi lên chuỗi prototype
dog.sound(); // Output: Animal makes a sound

// Kiểm tra liên kết:
console.log(dog.__proto__ === Animal.prototype); // Output: true
```

## 3. Kế Thừa Hiện Đại với ES6 Classes

ES6 (ECMAScript 2015) giới thiệu cú pháp `class`, giúp các lập trình viên quen thuộc với OOP truyền thống dễ dàng sử dụng hơn. Tuy nhiên, điều quan trọng cần nhớ là: `class` trong JavaScript chỉ là lớp **(Syntactic Sugar)** cho mô hình Kế thừa dựa trên Prototype.

```javascript
class Animal {
    constructor(name) {
        this.name = name;
    }

    sound() {
        console.log("Animal makes a sound");
    }
}

class Dog extends Animal { // extends thiết lập Prototype Chain
    bark() {
        console.log("Woof woof!");
    }
}

const myDog = new Dog("Buddy");
myDog.sound(); // Kế thừa từ Animal.prototype thông qua Prototype Chain
```

## Các từ khóa cần nhớ

- **Prototypal Inheritance**: Mô hình kế thừa của JS, đối tượng kế thừa trực tiếp từ đối tượng khác.
- **Prototype Chain**: Chuỗi các đối tượng prototype mà JS duyệt qua để tìm kiếm thuộc tính.
- **prototype**: Thuộc tính của hàm tạo/class, chứa các thuộc tính/phương thức để kế thừa.
- __proto__: Liên kết thực tế từ instance đến prototype của nó.
- **Syntactic Sugar**: Cú pháp `class` của ES6, giúp việc viết code kế thừa dễ đọc hơn.

## Vận dụng

- Tối ưu hóa bộ nhớ: Thay vì tạo lại phương thức `sound()` cho mỗi đối tượng **dog**, ta chỉ cần lưu nó một lần trên `Animal.prototype`.

Tạo các thư viện và framework sử dụng các mô hình kế thừa OOP rõ ràng.

## Tóm tắt kiến thức

Kế thừa trong JavaScript không dựa trên Class mà dựa trên Prototype. Mọi đối tượng JS đều có một liên kết (__proto__) đến đối tượng prototype của nó, tạo nên Prototype Chain. Cú pháp `class` hiện đại chỉ là một cách viết thân thiện hơn cho cơ chế cốt lõi này.

## Nguồn tham khảo

- [MDN Web Docs: Inheritance and the Prototype Chain](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Inheritance_and_the_prototype_chain)
- [Inheritance & Prototype Chain in Javascript](https://viblo.asia/p/inheritance-prototype-chain-in-javascript-YWOZryQEKQ0)

