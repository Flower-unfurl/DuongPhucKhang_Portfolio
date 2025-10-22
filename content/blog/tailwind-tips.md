---
title: "Tailwind CSS Tips & Tricks"
description: "Những mẹo nhỏ giúp bạn làm việc hiệu quả hơn với Tailwind CSS trong dự án Nuxt."
date: 2025-10-22
tags: ["Tailwind", "CSS", "Tips"]
cover: "/images/projects/worldmap.png"
draft: false
---

Tailwind CSS là một utility-first CSS framework mạnh mẽ. Dưới đây là một số mẹo giúp bạn sử dụng nó hiệu quả hơn.

## 1. Sử dụng @apply một cách thông minh

Mặc dù Tailwind khuyến khích dùng utility classes trực tiếp, nhưng `@apply` rất hữu ích cho các component tái sử dụng:

```css
.btn-primary {
  @apply bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-4 rounded;
}
```

## 2. Custom colors trong tailwind.config.js

Thêm màu sắc riêng của bạn vào theme:

```javascript
module.exports = {
  theme: {
    extend: {
      colors: {
        'brand-blue': '#011627',
        'brand-green': '#43D9AD',
      }
    }
  }
}
```

## 3. Responsive design với breakpoints

Tailwind có sẵn các breakpoints:

- `sm:` - 640px
- `md:` - 768px
- `lg:` - 1024px
- `xl:` - 1280px
- `2xl:` - 1536px

Ví dụ:

```html
<div class="text-sm md:text-base lg:text-lg">
  Responsive text size
</div>
```

## 4. Dark mode support

Tailwind 3+ hỗ trợ dark mode tuyệt vời:

```html
<div class="bg-white dark:bg-gray-800 text-black dark:text-white">
  Auto dark mode
</div>
```

## 5. JIT mode (Just-In-Time)

Tailwind 3 mặc định dùng JIT, cho phép:

- Generate classes on-demand
- Arbitrary values: `w-[137px]`, `top-[117px]`
- Faster build times

```html
<div class="mt-[137px] w-[calc(100%-2rem)]">
  Custom values
</div>
```

## Kết luận

Tailwind CSS kết hợp với Nuxt tạo ra trải nghiệm phát triển tuyệt vời. Hãy thử áp dụng các mẹo trên vào dự án của bạn!

> **Pro tip**: Cài đặt Tailwind CSS IntelliSense extension trong VS Code để có autocomplete và preview màu sắc.

Happy coding! 🎨
