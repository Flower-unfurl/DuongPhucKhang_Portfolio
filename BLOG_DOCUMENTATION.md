# Blog System Documentation

## Tổng quan

Blog được xây dựng với **Nuxt Content v3**, cho phép viết bài bằng Markdown và query như database.

## Cấu trúc thư mục

```
content/blog/
├── README.md                 # Hướng dẫn viết bài
├── hello-blog.md            # Bài giới thiệu
├── nuxt-content-co-ban.md   # Hướng dẫn Nuxt Content
└── tailwind-tips.md         # Tips Tailwind CSS
```

## Routes

- `/blog` - Trang danh sách tất cả bài viết
- `/blog/[slug]` - Trang chi tiết bài viết

## Components

### BlogCard.vue
Component hiển thị preview của một bài viết (ảnh, tiêu đề, mô tả, tags).

**Props:**
- `post: BlogPostCard` - Object chứa thông tin bài viết

**Usage:**
```vue
<BlogCard :post="post" />
```

## Pages

### /pages/blog/index.vue
Trang danh sách bài viết với các tính năng:

- **Search**: Tìm kiếm theo title/description
- **Filter by tag**: Lọc theo tag
- **Sort**: Sắp xếp theo ngày (mới nhất/cũ nhất)
- **Pagination**: 9 bài/trang
- **Responsive grid**: 1 cột (mobile), 2 cột (tablet), 3 cột (desktop)

### /pages/blog/[slug].vue
Trang chi tiết bài viết:

- Cover image (16:9)
- Metadata (ngày, thời gian đọc, tags)
- Nội dung Markdown được render
- Table of contents (desktop, sticky sidebar)
- Styling cho các element Markdown (headings, code, links, etc.)

## Cách sử dụng

### 1. Thêm bài viết mới

Tạo file `.md` trong `content/blog/`:

```markdown
---
title: "Tiêu đề bài viết"
description: "Mô tả ngắn"
date: 2025-10-22
tags: ["Tag1", "Tag2"]
cover: "/images/blog/cover.png"
draft: false
---

Nội dung bài viết...
```

### 2. Draft mode

Set `draft: true` để ẩn bài viết khỏi danh sách.

### 3. Thêm ảnh

- Đặt ảnh trong `public/images/blog/`
- Reference trong frontmatter: `cover: "/images/blog/filename.png"`
- Trong nội dung: `![Alt text](/images/blog/image.png)`

### 4. Code highlighting

Nuxt Content tự động highlight code:

````markdown
```javascript
const hello = 'world'
```
````

### 5. Tags

Tags được tự động thu thập từ tất cả bài viết và hiển thị dưới dạng filter pills.

## Styling

### Blog content styling

Các element trong bài viết được style tự động:

- **Headings**: Bold, margins, scroll-margin-top
- **Links**: Purple color, hover underline
- **Code**: Background, color, padding
- **Blockquotes**: Border-left, italic
- **Tables**: Borders, padding
- **Images**: Rounded corners

### Color scheme

- Background: `#011221`
- Border: `#1E2D3D`
- Text: `#85a5c4` (menu-text)
- Highlight: `#607B96`
- Accent: `#799ffb` (purplefy)
- Code: `#43D9AD` (greenfy)

## TypeScript Interfaces

```typescript
interface BlogPost {
  _path: string
  title: string
  description?: string
  date: string
  tags?: string[]
  cover?: string
  draft?: boolean
  body?: {
    toc?: {
      links?: TocLink[]
    }
  }
}
```

## Performance

- **Lazy loading images**: `loading="lazy"` attribute
- **Code splitting**: Automatic per-route
- **Static generation**: Can use `nuxt generate` for SSG
- **Caching**: useAsyncData with keys

## SEO

Mỗi trang tự động có:

- Dynamic `<title>`
- Meta description
- Có thể thêm: og:image, Twitter cards

## Tính năng có thể mở rộng

- [ ] View counter
- [ ] Reading progress bar
- [ ] Related posts
- [ ] Previous/Next navigation
- [ ] Comments (Giscus/Utterances)
- [ ] RSS feed
- [ ] Series/categories
- [ ] Search với Algolia/Fuse.js
- [ ] Share buttons
- [ ] Reading time estimator (đã có cơ bản)

## Troubleshooting

### queryContent not found
- Chạy `yarn postinstall` để generate types
- Restart TypeScript server trong VS Code

### Styling không apply
- Check tailwind.config.js
- Verify @nuxtjs/tailwindcss module enabled

### Bài viết không hiển thị
- Check frontmatter format (YAML)
- Verify `draft: false`
- Check file nằm trong `content/blog/`

## Resources

- [Nuxt Content Docs](https://content.nuxt.com/)
- [Tailwind CSS Docs](https://tailwindcss.com/)
- [Markdown Guide](https://www.markdownguide.org/)
