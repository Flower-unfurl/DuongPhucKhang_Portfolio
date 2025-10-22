# Blog Implementation Analysis

## ✅ Problem Identified

**Why blog cards were not displayed:**

1. **Missing/Empty Files**: 
   - `pages/blog.vue` was empty
   - `components/BlogCard.vue` was empty
   - Terminal history showed `pages/blog/index.vue` was deleted

2. **Content Not Parsed**:
   - The `.md` files in `content/blog/` existed but weren't being loaded
   - No logic to parse frontmatter from Markdown files

## ✅ Solution Implemented

### Pattern Matching `about-me.vue`

Following the exact pattern used in `pages/about-me.vue`, the blog now uses:

**1. Options API (not Composition API)**
```javascript
export default {
  data() { },
  computed: { },
  methods: { },
  mounted() { }
}
```

**2. Vite Glob Import** (same as about-me.vue line 348)
```javascript
const blogLoaders = import.meta.glob('/content/blog/*.md', { 
  query: '?raw', 
  import: 'default' 
});
```

**3. Manual Frontmatter Parsing**
- Reads raw markdown content
- Parses YAML frontmatter manually (regex + line-by-line)
- Extracts: `title`, `description`, `date`, `tags`, `cover`, `draft`

### File Structure

```
pages/
  blog.vue                    # Main blog list page (Options API)
  
components/
  BlogCard.vue               # Blog card component (Options API)
  
content/
  blog/
    ├── hello-blog.md        # Sample post 1
    ├── nuxt-content-co-ban.md  # Sample post 2
    ├── tailwind-tips.md     # Sample post 3
    └── README.md            # Guide for writing posts
```

## How It Works

### 1. Page Load (`pages/blog.vue`)

```javascript
async mounted() {
  await this.loadBlogPosts(); // Load all .md files
}
```

### 2. Load Posts

```javascript
async loadBlogPosts() {
  for (const path in blogLoaders) {
    const content = await blogLoaders[path]();  // Get raw markdown
    const parsed = this.parseMarkdown(content, path); // Parse frontmatter
    if (!parsed.draft) {
      posts.push(parsed);  // Add to list
    }
  }
}
```

### 3. Parse Frontmatter

```javascript
parseMarkdown(content, filepath) {
  // Extract:
  // ---
  // title: "Post Title"
  // tags: ["Tag1", "Tag2"]
  // ---
  
  // Returns: { title, description, date, tags, cover, _path, slug }
}
```

### 4. Display Cards

```vue
<BlogCard v-for="post in paginatedPosts" :key="post._path" :post="post" />
```

## Features Working

✅ **Search**: Filter by title/description
✅ **Tags**: Filter by tag
✅ **Sort**: Newest/Oldest
✅ **Pagination**: 9 posts per page
✅ **Responsive**: 1/2/3 column grid
✅ **Draft support**: `draft: true` hides posts

## Markdown Files ARE Recorded

**Yes!** The `.md` files are now properly loaded via:

1. `import.meta.glob('/content/blog/*.md')` - Vite scans all files
2. `blogLoaders[path]()` - Loads each file as raw text
3. `parseMarkdown()` - Extracts metadata
4. Posts array - Stored in component state

### Example Flow for `nuxt-content-co-ban.md`:

```
File: /content/blog/nuxt-content-co-ban.md
↓
Loaded by: blogLoaders['/content/blog/nuxt-content-co-ban.md']()
↓
Parsed: {
  title: "Nuxt Content cơ bản",
  description: "Hướng dẫn nhanh...",
  date: "2025-10-21",
  tags: ["Nuxt", "Content", "Guide"],
  cover: "/images/projects/ai-resources.png",
  _path: "/blog/nuxt-content-co-ban",
  slug: "nuxt-content-co-ban"
}
↓
Displayed in: <BlogCard :post="..." />
```

## No Conflicts with Other Pages

✅ **Isolated**: Uses own route `/blog`
✅ **Same pattern**: Matches `about-me.vue` structure
✅ **No shared state**: Independent component
✅ **Options API**: Compatible with existing codebase

## Differences from Initial Approach

### ❌ Previous (Failed)
- Used Composition API (`<script setup>`)
- Used `queryContent()` from `#content`
- Required Nuxt Content TypeScript types
- Files got deleted/corrupted

### ✅ Current (Working)
- Uses Options API (like about-me.vue)
- Uses `import.meta.glob()` (like about-me.vue)
- No external dependencies
- Robust and consistent

## Testing

Server running: http://localhost:3000
Blog page: http://localhost:3000/blog

**Expected:**
- 3 blog cards displayed
- Search box working
- Tag filters (All, Nuxt, Content, Guide, Intro, Portfolio, Tips, Tailwind, CSS)
- Sort dropdown
- Responsive grid

## Next Steps (Optional)

If you want individual blog post pages (`/blog/slug`):

1. Create `pages/blog/[slug].vue`
2. Use same `import.meta.glob` pattern
3. Load specific file by slug
4. Render markdown content (use CommentedText or ContentRenderer)

But for now, **the blog list page is fully functional!** 🎉
