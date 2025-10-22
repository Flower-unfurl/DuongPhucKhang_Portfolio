# Blog Content

Thư mục này chứa các bài viết blog được viết bằng Markdown.

## Cấu trúc file

Mỗi bài viết cần có frontmatter (phần metadata ở đầu file) như sau:

```markdown
---
title: "Tiêu đề bài viết"
description: "Mô tả ngắn gọn về bài viết"
date: 2025-10-22
tags: ["Tag1", "Tag2", "Tag3"]
cover: "/images/blog/cover-image.png"
draft: false
---

Nội dung bài viết ở đây...
```

## Các field bắt buộc

- `title`: Tiêu đề bài viết
- `description`: Mô tả ngắn (hiển thị trong card)
- `date`: Ngày xuất bản (format: YYYY-MM-DD)
- `tags`: Mảng các tag (để filter)
- `cover`: Đường dẫn ảnh cover
- `draft`: `false` để xuất bản, `true` để ẩn

## Cách viết bài mới

1. Tạo file mới trong thư mục này: `ten-bai-viet.md`
2. Thêm frontmatter ở đầu file
3. Viết nội dung bằng Markdown
4. Save và xem tại `/blog`

## Markdown features

- **Headings**: `# H1`, `## H2`, `### H3`
- **Bold**: `**text**`
- **Italic**: `*text*`
- **Links**: `[text](url)`
- **Images**: `![alt](url)`
- **Code inline**: `` `code` ``
- **Code blocks**: 
  ````
  ```javascript
  const code = 'here'
  ```
  ````
- **Lists**: `-` hoặc `1.`
- **Blockquotes**: `> quote`

## Tips

- Đặt tên file bằng slug (lowercase, dấu gạch ngang)
- Cover image nên có tỷ lệ 16:9
- Dùng tags nhất quán để dễ filter
- Viết description ngắn gọn (2-3 câu)
