---
title: "Nuxt Content cơ bản"
description: "Hướng dẫn nhanh cách tạo blog với Nuxt Content: cấu trúc nội dung, routing, và render Markdown."
date: 2025-10-21
tags: ["Nuxt", "Content", "Guide"]
cover: "/images/projects/ai-resources.png"
draft: false
---

Nuxt Content giúp bạn viết bài bằng Markdown nhưng vẫn truy vấn linh hoạt như cơ sở dữ liệu.

### 1) Tạo file Markdown

Đặt bài viết trong `content/blog/ten-bai.md` với phần frontmatter chứa `title, description, date, tags, cover`.

### 2) Liệt kê bài viết

Sử dụng `queryContent('/blog')` để lọc, sắp xếp, phân trang bài viết.

### 3) Trang chi tiết

Dùng `<ContentDoc :path="`/blog/${slug}`" />` hoặc truy vấn và `<ContentRenderer :value="doc" />`.

Chỉ vậy thôi, bạn đã có blog tối giản và dễ mở rộng! ✨
