<template>
  <main class="flex flex-col flex-auto overflow-hidden">
    <div id="mobile-page-title">
      <h2>_blog</h2>
    </div>

    <!-- Desktop tab -->
    <div class="w-full border-bot hidden lg:flex items-center tab-height">
      <div class="flex items-center border-right h-full">
        <p class="font-roboto_mono_regular text-menu-text text-sm px-3">blog</p>
      </div>
    </div>

    <!-- Content area -->
    <section class="w-full h-full overflow-y-auto px-6 lg:px-12 py-6">
      <!-- Search and filters -->
      <div class="flex flex-col gap-4 mb-6">
        <div class="flex items-center gap-3 flex-wrap">
          <input 
            v-model="searchQuery" 
            type="search" 
            placeholder="Tìm kiếm bài viết..." 
            class="bg-transparent border border-[#1E2D3D] rounded-lg px-3 py-2 text-sm text-white w-full max-w-xl placeholder-[#465E77] focus:outline-none focus:border-[#607B96] font-roboto_mono_regular" 
          />
          <select 
            v-model="sortOrder" 
            class="bg-[#011221] border border-[#1E2D3D] rounded-lg px-3 py-2 text-sm text-menu-text focus:outline-none focus:border-[#607B96] font-roboto_mono_regular"
          >
            <option value="date-desc">Mới nhất</option>
            <option value="date-asc">Cũ nhất</option>
          </select>
        </div>
        
        <div class="flex items-center gap-2 overflow-x-auto pb-1">
          <button 
            @click="selectedTag = 'all'" 
            :class="tagButtonClass('all')"
          >
            All
          </button>
          <button 
            v-for="tag in availableTags" 
            :key="tag" 
            @click="selectedTag = tag" 
            :class="tagButtonClass(tag)"
          >
            #{{ tag }}
          </button>
        </div>
      </div>

      <!-- Blog grid -->
      <div v-if="filteredPosts.length > 0" class="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-3 gap-6">
        <BlogCard v-for="post in paginatedPosts" :key="post._path" :post="post" />
      </div>

      <!-- Empty state -->
      <div v-else class="flex flex-col items-center justify-center py-20 text-menu-text">
        <p class="text-4xl mb-4">¯\_(ツ)_/¯</p>
        <p class="text-lg text-white">Không tìm thấy bài viết</p>
        <p class="text-sm">Thử thay đổi bộ lọc hoặc từ khóa tìm kiếm</p>
      </div>

      <!-- Pagination -->
      <div v-if="totalPages > 1" class="flex items-center justify-center gap-3 mt-8">
        <button 
          class="px-4 py-2 rounded-lg border border-[#1E2D3D] text-menu-text disabled:opacity-40 hover:border-[#607B96] transition font-roboto_mono_regular" 
          :disabled="currentPage === 1" 
          @click="currentPage--"
        >
          ← Trước
        </button>
        <span class="text-menu-text text-sm font-roboto_mono_regular">
          Trang {{ currentPage }} / {{ totalPages }}
        </span>
        <button 
          class="px-4 py-2 rounded-lg border border-[#1E2D3D] text-menu-text disabled:opacity-40 hover:border-[#607B96] transition font-roboto_mono_regular" 
          :disabled="currentPage === totalPages" 
          @click="currentPage++"
        >
          Sau →
        </button>
      </div>
    </section>
  </main>
</template>

<script>
// Load all blog markdown files using Vite glob import
const blogLoaders = import.meta.glob('/content/blog/*.md', { 
  query: '?raw', 
  import: 'default' 
});

export default {
  data() {
    return {
      posts: [],
      searchQuery: '',
      selectedTag: 'all',
      sortOrder: 'date-desc',
      currentPage: 1,
      pageSize: 9,
      loading: true
    }
  },
  
  async mounted() {
    await this.loadBlogPosts();
  },
  
  computed: {
    // Extract unique tags from all posts
    availableTags() {
      const tagSet = new Set();
      this.posts.forEach(post => {
        if (post.tags && Array.isArray(post.tags)) {
          post.tags.forEach(tag => tagSet.add(tag));
        }
      });
      return Array.from(tagSet).sort();
    },
    
    // Filter posts based on search and tag
    filteredPosts() {
      let filtered = [...this.posts];
      
      // Filter by tag
      if (this.selectedTag !== 'all') {
        filtered = filtered.filter(post => 
          post.tags && post.tags.includes(this.selectedTag)
        );
      }
      
      // Filter by search query
      if (this.searchQuery.trim()) {
        const query = this.searchQuery.toLowerCase();
        filtered = filtered.filter(post =>
          (post.title && post.title.toLowerCase().includes(query)) ||
          (post.description && post.description.toLowerCase().includes(query))
        );
      }
      
      // Sort by date
      filtered.sort((a, b) => {
        const dateA = new Date(a.date).getTime();
        const dateB = new Date(b.date).getTime();
        return this.sortOrder === 'date-desc' ? dateB - dateA : dateA - dateB;
      });
      
      return filtered;
    },
    
    // Calculate total pages
    totalPages() {
      return Math.max(1, Math.ceil(this.filteredPosts.length / this.pageSize));
    },
    
    // Get posts for current page
    paginatedPosts() {
      const start = (this.currentPage - 1) * this.pageSize;
      return this.filteredPosts.slice(start, start + this.pageSize);
    }
  },
  
  watch: {
    // Reset to page 1 when filters change
    searchQuery() {
      this.currentPage = 1;
    },
    selectedTag() {
      this.currentPage = 1;
    },
    sortOrder() {
      this.currentPage = 1;
    }
  },
  
  methods: {
    // Load all blog posts from markdown files
    async loadBlogPosts() {
      this.loading = true;
      const posts = [];
      
      for (const path in blogLoaders) {
        try {
          const content = await blogLoaders[path]();
          const parsed = this.parseMarkdown(content, path);
          
          // Only include non-draft posts
          if (!parsed.draft) {
            posts.push(parsed);
          }
        } catch (error) {
          console.error('Error loading blog post:', path, error);
        }
      }
      
      this.posts = posts;
      this.loading = false;
    },
    
    // Parse markdown frontmatter and content
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
        draft: false
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
      }
      
      return post;
    },
    
    // Generate CSS class for tag buttons
    tagButtonClass(tag) {
      const isActive = this.selectedTag === tag;
      return [
        'text-xs whitespace-nowrap border rounded-md px-3 py-1.5 transition font-roboto_mono_regular',
        isActive 
          ? 'text-white border-[#607B96] bg-[#1C2B3A]' 
          : 'text-menu-text border-[#1E2D3D] hover:border-[#607B96] hover:text-white'
      ];
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
</style>