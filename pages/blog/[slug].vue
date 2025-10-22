<template>
  <main class="flex flex-col flex-auto overflow-hidden">
    <div id="mobile-page-title">
      <h2>_blog</h2>
    </div>

    <!-- Desktop tab -->
    <div class="w-full border-bot hidden lg:flex items-center tab-height">
      <div class="flex items-center border-right h-full">
        <p class="font-roboto_mono_regular text-menu-text text-sm px-3">{{ post?.title || 'Blog' }}</p>
      </div>
    </div>

    <!-- Loading state -->
    <section v-if="loading" class="w-full h-full overflow-y-auto px-6 lg:px-12 py-6 flex items-center justify-center">
      <div class="text-menu-text text-center">
        <p class="text-2xl mb-2">⏳</p>
        <p>Đang tải...</p>
      </div>
    </section>

    <!-- Error state -->
    <section v-else-if="error" class="w-full h-full overflow-y-auto px-6 lg:px-12 py-6">
      <div class="text-center py-20">
        <p class="text-4xl mb-4 text-menu-text">404</p>
        <h1 class="text-2xl text-white mb-4">Không tìm thấy bài viết</h1>
        <p class="text-menu-text mb-6">Bài viết này không tồn tại hoặc đã bị xóa.</p>
        <NuxtLink 
          to="/blog" 
          class="inline-block px-6 py-3 bg-[#1C2B3A] border border-[#607B96] text-white rounded-lg hover:bg-[#263B4A] transition font-roboto_mono_regular"
        >
          ← Quay về trang Blog
        </NuxtLink>
      </div>
    </section>

    <!-- Content area -->
    <section v-else class="w-full h-full overflow-y-auto px-6 lg:px-12 py-6">
      <article class="max-w-4xl mx-auto">
        <!-- Back button -->
        <NuxtLink 
          to="/blog" 
          class="inline-flex items-center gap-2 text-menu-text hover:text-white transition mb-6 font-roboto_mono_regular text-sm"
        >
          <span>←</span>
          <span>Quay lại</span>
        </NuxtLink>

        <!-- Cover image -->
        <div v-if="post.cover" class="relative aspect-[21/9] overflow-hidden bg-[#1E2D3D] rounded-xl mb-8">
          <img 
            :src="post.cover" 
            :alt="post.title" 
            class="w-full h-full object-cover" 
          />
        </div>

        <!-- Post header -->
        <header class="mb-8">
          <h1 class="text-3xl lg:text-4xl text-white font-roboto_mono_bold mb-4">
            {{ post.title }}
          </h1>

          <p v-if="post.description" class="text-lg text-menu-text font-roboto_mono_regular mb-4">
            {{ post.description }}
          </p>

          <!-- Meta info -->
          <div class="flex flex-wrap items-center gap-4 text-sm text-menu-text font-roboto_mono_regular">
            <span v-if="post.date">📅 {{ formatDate(post.date) }}</span>
            <span v-if="post.readingTime">⏱️ {{ post.readingTime }} phút đọc</span>
          </div>

          <!-- Tags -->
          <div v-if="post.tags && post.tags.length" class="flex flex-wrap gap-2 mt-4">
            <NuxtLink 
              v-for="tag in post.tags" 
              :key="tag" 
              :to="`/blog?tag=${tag}`"
              class="text-xs text-menu-text border border-[#1E2D3D] rounded-md px-3 py-1.5 hover:border-[#607B96] hover:text-white transition"
            >
              #{{ tag }}
            </NuxtLink>
          </div>
        </header>

        <!-- Post content -->
        <div class="prose prose-invert max-w-none" v-html="renderedContent"></div>

        <!-- Footer navigation -->
        <footer class="mt-12 pt-8 border-t border-[#1E2D3D]">
          <NuxtLink 
            to="/blog" 
            class="inline-flex items-center gap-2 text-menu-text hover:text-white transition font-roboto_mono_regular"
          >
            <span>←</span>
            <span>Xem tất cả bài viết</span>
          </NuxtLink>
        </footer>
      </article>
    </section>
  </main>
</template>

<script>
import { marked } from 'marked';

// Configure marked options
marked.setOptions({
  breaks: true,
  gfm: true,
  headerIds: true,
  mangle: false
});

// Load all blog markdown files using Vite glob import (same as index.vue)
const blogLoaders = import.meta.glob('/content/blog/*.md', { 
  query: '?raw', 
  import: 'default' 
});

export default {
  data() {
    return {
      post: null,
      loading: true,
      error: false
    }
  },
  
  computed: {
    renderedContent() {
      if (!this.post || !this.post.body) return '';
      return marked(this.post.body);
    }
  },
  
  async mounted() {
    await this.loadBlogPost();
  },
  
  methods: {
    // Load specific blog post by slug
    async loadBlogPost() {
      this.loading = true;
      this.error = false;
      
      try {
        // Get slug from route params
        const slug = this.$route.params.slug;
        
        if (!slug) {
          this.error = true;
          this.loading = false;
          return;
        }
        
        // Find the matching file
        const filepath = `/content/blog/${slug}.md`;
        
        if (!blogLoaders[filepath]) {
          console.error('Blog post not found:', filepath);
          this.error = true;
          this.loading = false;
          return;
        }
        
        // Load and parse the markdown file
        const content = await blogLoaders[filepath]();
        this.post = this.parseMarkdown(content, filepath);
        
        // Check if post is draft
        if (this.post.draft) {
          this.error = true;
          this.loading = false;
          return;
        }
        
        this.loading = false;
      } catch (err) {
        console.error('Error loading blog post:', err);
        this.error = true;
        this.loading = false;
      }
    },
    
    // Parse markdown frontmatter and content (same as index.vue)
    parseMarkdown(content, filepath) {
      // Extract slug from filepath
      const slug = filepath.split('/').pop().replace('.md', '');
      
      // Parse frontmatter
      const frontmatterRegex = /^---\n([\s\S]*?)\n---/;
      const match = content.match(frontmatterRegex);
      
      const post = {
        _path: `/blog/${slug}`,
        slug: slug,
        title: slug,
        description: '',
        date: new Date().toISOString().split('T')[0],
        tags: [],
        cover: '/images/projects/ui-animations2.png',
        draft: false,
        body: '' // Will contain the actual content
      };
      
      if (match) {
        const frontmatter = match[1];
        const lines = frontmatter.split('\n');
        
        let currentKey = null;
        let currentArray = [];
        
        lines.forEach(line => {
          // Handle key-value pairs
          if (line.includes(':') && !line.trim().startsWith('-')) {
            // Save previous array if exists
            if (currentKey && currentArray.length > 0) {
              post[currentKey] = currentArray;
              currentArray = [];
            }
            
            const [key, ...valueParts] = line.split(':');
            const value = valueParts.join(':').trim();
            currentKey = key.trim();
            
            if (value) {
              // Remove quotes if present
              const cleanValue = value.replace(/^["']|["']$/g, '');
              post[currentKey] = cleanValue;
              currentKey = null;
            }
          } 
          // Handle array items
          else if (line.trim().startsWith('-') && currentKey) {
            const item = line.trim().substring(1).trim().replace(/^["']|["']$/g, '');
            if (item) {
              currentArray.push(item);
            }
          }
        });
        
        // Save last array if exists
        if (currentKey && currentArray.length > 0) {
          post[currentKey] = currentArray;
        }
        
        // Convert string boolean to actual boolean
        if (typeof post.draft === 'string') {
          post.draft = post.draft.toLowerCase() === 'true';
        }
        
        // Extract body content (everything after frontmatter)
        post.body = content.substring(match[0].length).trim();
        
        // Calculate reading time (rough estimate: 200 words per minute)
        const wordCount = post.body.split(/\s+/).length;
        post.readingTime = Math.ceil(wordCount / 200);
      } else {
        // No frontmatter, use entire content as body
        post.body = content;
      }
      
      return post;
    },
    
    // Format date
    formatDate(dateString) {
      if (!dateString) return '';
      
      try {
        const date = new Date(dateString);
        return date.toLocaleDateString('vi-VN', {
          year: 'numeric',
          month: 'long',
          day: 'numeric'
        });
      } catch (error) {
        return dateString;
      }
    }
  }
}
</script>

<style scoped>
.tab-height {
  min-height: 35px;
  max-height: 35px;
}

#mobile-page-title {
  display: flex;
  font-size: 14px;
  height: 70px;
  color: white;
  padding: 0 25px;
  align-items: center;
}

@media (min-width: 1024px) {
  #mobile-page-title {
    display: none;
  }
}

/* Prose styles for markdown content */
.prose {
  color: #607B96;
  line-height: 1.8;
  font-size: 1rem;
}

.prose :deep(h1) {
  color: white;
  font-size: 2rem;
  font-weight: bold;
  margin-top: 2.5rem;
  margin-bottom: 1.25rem;
  font-family: 'Roboto Mono', monospace;
  line-height: 1.2;
}

.prose :deep(h2) {
  color: white;
  font-size: 1.75rem;
  font-weight: bold;
  margin-top: 2rem;
  margin-bottom: 1rem;
  font-family: 'Roboto Mono', monospace;
  line-height: 1.3;
  border-bottom: 1px solid #1E2D3D;
  padding-bottom: 0.5rem;
}

.prose :deep(h3) {
  color: white;
  font-size: 1.5rem;
  font-weight: bold;
  margin-top: 1.5rem;
  margin-bottom: 0.75rem;
  font-family: 'Roboto Mono', monospace;
  line-height: 1.4;
}

.prose :deep(h4) {
  color: white;
  font-size: 1.25rem;
  font-weight: bold;
  margin-top: 1.25rem;
  margin-bottom: 0.5rem;
  font-family: 'Roboto Mono', monospace;
}

.prose :deep(p) {
  margin-top: 1rem;
  margin-bottom: 1rem;
  font-family: 'Roboto Mono', monospace;
  line-height: 1.8;
}

.prose :deep(strong) {
  color: #E5E9F0;
  font-weight: 700;
}

.prose :deep(em) {
  font-style: italic;
  color: #C8D0DD;
}

.prose :deep(a) {
  color: #4D5BCE;
  text-decoration: underline;
  transition: color 0.2s;
}

.prose :deep(a:hover) {
  color: #7380EC;
}

.prose :deep(ul), .prose :deep(ol) {
  margin-top: 1rem;
  margin-bottom: 1rem;
  padding-left: 2rem;
}

.prose :deep(ul) {
  list-style-type: disc;
}

.prose :deep(ol) {
  list-style-type: decimal;
}

.prose :deep(li) {
  margin-top: 0.5rem;
  margin-bottom: 0.5rem;
  line-height: 1.7;
}

.prose :deep(li > p) {
  margin: 0.25rem 0;
}

.prose :deep(code) {
  background-color: #1E2D3D;
  padding: 0.2rem 0.5rem;
  border-radius: 0.25rem;
  font-size: 0.875rem;
  color: #43D9AD;
  font-family: 'Roboto Mono', monospace;
}

.prose :deep(pre) {
  background-color: #011221;
  border: 1px solid #1E2D3D;
  border-radius: 0.5rem;
  padding: 1.25rem;
  overflow-x: auto;
  margin-top: 1.5rem;
  margin-bottom: 1.5rem;
}

.prose :deep(pre code) {
  background-color: transparent;
  padding: 0;
  color: #E5E9F0;
  font-size: 0.875rem;
  line-height: 1.6;
}

.prose :deep(blockquote) {
  border-left: 4px solid #607B96;
  padding-left: 1.5rem;
  padding-top: 0.5rem;
  padding-bottom: 0.5rem;
  font-style: italic;
  color: #8B9BB0;
  margin-top: 1.5rem;
  margin-bottom: 1.5rem;
  background-color: rgba(96, 123, 150, 0.05);
}

.prose :deep(blockquote p) {
  margin: 0.5rem 0;
}

.prose :deep(img) {
  border-radius: 0.5rem;
  margin-top: 1.5rem;
  margin-bottom: 1.5rem;
  max-width: 100%;
  height: auto;
}

.prose :deep(hr) {
  border: none;
  border-top: 1px solid #1E2D3D;
  margin-top: 2rem;
  margin-bottom: 2rem;
}

.prose :deep(table) {
  width: 100%;
  border-collapse: collapse;
  margin-top: 1.5rem;
  margin-bottom: 1.5rem;
  font-size: 0.875rem;
}

.prose :deep(th), .prose :deep(td) {
  border: 1px solid #1E2D3D;
  padding: 0.75rem;
  text-align: left;
}

.prose :deep(th) {
  background-color: #1E2D3D;
  color: white;
  font-weight: bold;
}

.prose :deep(tr:nth-child(even)) {
  background-color: rgba(30, 45, 61, 0.3);
}

/* Syntax highlighting for code blocks */
.prose :deep(pre code.language-java),
.prose :deep(pre code.language-javascript),
.prose :deep(pre code.language-python) {
  display: block;
  overflow-x: auto;
}

/* Scrollbar styling for code blocks */
.prose :deep(pre)::-webkit-scrollbar {
  height: 8px;
}

.prose :deep(pre)::-webkit-scrollbar-track {
  background: #011221;
}

.prose :deep(pre)::-webkit-scrollbar-thumb {
  background: #1E2D3D;
  border-radius: 4px;
}

.prose :deep(pre)::-webkit-scrollbar-thumb:hover {
  background: #607B96;
}
</style>
