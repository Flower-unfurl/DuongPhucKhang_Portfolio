<template>
  <main class="flex flex-col flex-auto lg:flex-row overflow-hidden">
    <div id="mobile-page-title">
      <h2>_blog</h2>
    </div>

    <!-- section title (mobile) -->
    <div id="section-content-title" class="flex lg:hidden" @click="showFilters = !showFilters">
      <img :class="showFilters ? 'section-arrow rotate-90' : 'section-arrow'" src="/icons/arrow.svg">
      <span class="font-roboto_mono_regular text-white text-sm">blog-posts</span>
    </div>

    <div v-if="showFilters" id="filter-menu"
      class="w-full flex-col border-right font-roboto_mono_regular text-menu-text lg:flex">
      <!-- title -->
      <div id="section-content-title" class="hidden lg:flex items-center min-w-full">
        <img id="section-arrow-menu" src="/icons/arrow.svg" alt="" class="section-arrow mx-3">
        <p class="font-roboto_mono_regular text-white text-sm">blog-posts</p>
      </div>

      <!-- filter menu -->
      <nav id="filters" class="w-full flex-col">
        <!-- Search box -->
        <div class="mb-4">
          <input 
            v-model="searchQuery" 
            type="search" 
            placeholder="Tìm kiếm..." 
            class="w-full bg-transparent border border-[#1E2D3D] rounded-lg px-3 py-2 text-sm text-white placeholder-[#465E77] focus:outline-none focus:border-[#607B96] font-roboto_mono_regular" 
          />
        </div>

        <!-- Sort order -->
        <div class="mb-4">
          <select 
            v-model="sortOrder" 
            class="w-full bg-[#011221] border border-[#1E2D3D] rounded-lg px-3 py-2 text-sm text-menu-text focus:outline-none focus:border-[#607B96] font-roboto_mono_regular"
          >
            <option value="date-desc">Mới nhất</option>
            <option value="date-asc">Cũ nhất</option>
          </select>
        </div>

        <!-- Tag filters -->
        <div class="flex flex-col">
          <div class="flex items-center py-2 cursor-pointer" @click="filterByTag('all')">
            <input type="checkbox" id="tag-all" :checked="selectedTag === 'all'">
            <label for="tag-all" :class="selectedTag === 'all' ? 'text-white ml-4' : 'ml-4'">All</label>
          </div>

          <div v-for="tag in availableTags" :key="tag" class="flex items-center py-2 cursor-pointer" @click="filterByTag(tag)">
            <input type="checkbox" :id="'tag-' + tag" :checked="selectedTag === tag">
            <label :for="'tag-' + tag" :class="selectedTag === tag ? 'text-white ml-4' : 'ml-4'">#{{ tag }}</label>
          </div>
        </div>
      </nav>
    </div>

    <!-- content -->
    <div class="flex flex-col w-full overflow-hidden">
      <!-- windows tab -->
      <div class="tab-height w-full hidden lg:flex border-bot items-center">
        <div class="flex items-center border-right h-full">
          <p class="font-roboto_mono_regular text-menu-text text-sm px-3">{{ selectedTag === 'all' ? 'all' : selectedTag }};</p>
          <img src="/icons/close.svg" alt="" class="m-3">
        </div>
      </div>

      <!-- windows tab mobile -->
      <div id="tab" class="flex lg:hidden items-center">
        <span class="text-white"> // </span>
        <p class="font-roboto_mono_regular text-white text-sm px-3">blog</p>
        <span class="text-menu-text"> / </span>
        <p class="font-roboto_mono_regular text-menu-text text-sm px-3">{{ selectedTag === 'all' ? 'all' : selectedTag }};</p>
      </div>

      <!-- blog posts -->
      <div id="blog-posts-case" class="grid grid-cols-1 lg:grid-cols-2 max-w-full h-full overflow-scroll lg:self-center">
        <div id="not-found"
          class="hidden flex flex-col font-roboto_mono_regular text-menu-text my-5 h-full justify-center items-center">
          <span class="flex justify-center text-4xl pb-3">
            ¯\_(ツ)_/¯
          </span>
          <span class="text-white flex justify-center text-xl">
            Không tìm thấy bài viết
          </span>
          <span class="flex justify-center">
            Thử thay đổi bộ lọc hoặc từ khóa tìm kiếm
          </span>
        </div>

        <BlogCard v-for="(post, index) in paginatedPosts" :key="post._path" :index="index" :post="post" />
      </div>

      <!-- Pagination -->
      <div v-if="totalPages > 1" class="flex items-center justify-center gap-3 py-4 border-top">
        <button 
          class="px-4 py-2 rounded-lg border border-[#1E2D3D] text-menu-text disabled:opacity-40 hover:border-[#607B96] transition font-roboto_mono_regular text-xs" 
          :disabled="currentPage === 1" 
          @click="currentPage--"
        >
          ← Trước
        </button>
        <span class="text-menu-text text-sm font-roboto_mono_regular">
          {{ currentPage }} / {{ totalPages }}
        </span>
        <button 
          class="px-4 py-2 rounded-lg border border-[#1E2D3D] text-menu-text disabled:opacity-40 hover:border-[#607B96] transition font-roboto_mono_regular text-xs" 
          :disabled="currentPage === totalPages" 
          @click="currentPage++"
        >
          Sau →
        </button>
      </div>
    </div>
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
      pageSize: 6,
      loading: true,
      showFilters: true
    }
  },
  
  async mounted() {
    await this.loadBlogPosts();
    // If there's a tag in query params, preselect it
    try {
      const urlParams = new URLSearchParams(window.location.search)
      const tag = urlParams.get('tag')
      if (tag) this.selectedTag = tag
    } catch (e) {
      // ignore
    }
    this.updateNotFoundState();
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
      this.$nextTick(() => this.updateNotFoundState());
    },
    selectedTag() {
      this.currentPage = 1;
      this.$nextTick(() => this.updateNotFoundState());
    },
    sortOrder() {
      this.currentPage = 1;
      this.$nextTick(() => this.updateNotFoundState());
    }
  },
  
  methods: {
    // Load all blog posts from markdown files
    async loadBlogPosts() {
      this.loading = true;
      const posts = [];
      
      for (const path in blogLoaders) {
        try {
          // Skip README and files starting with underscore
          const filename = path.split('/').pop();
          if (filename.startsWith('_') || filename.toLowerCase().startsWith('readme')) {
            continue;
          }
          
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
    
    // Filter by tag (similar to filterProjects in projects.vue)
    filterByTag(tag) {
      this.selectedTag = tag;
      // Update URL query param
      if (typeof window !== 'undefined') {
        const url = new URL(window.location.href)
        url.searchParams.set('tag', tag)
        window.history.replaceState({}, '', url.toString())
      }
    },

    // Update not found state
    updateNotFoundState() {
      if (this.filteredPosts.length === 0) {
        const postsCase = document.getElementById('blog-posts-case');
        const notFound = document.getElementById('not-found');
        if (postsCase && notFound) {
          postsCase.classList.remove('grid');
          notFound.classList.remove('hidden');
        }
      } else {
        const postsCase = document.getElementById('blog-posts-case');
        const notFound = document.getElementById('not-found');
        if (postsCase && notFound) {
          postsCase.classList.add('grid');
          notFound.classList.add('hidden');
        }
      }
    }
  }
}
</script>

<style scoped>
#filters {
  padding: 10px 25px;
}

#tab {
  padding: 25px 25px 5px;
  flex-wrap: wrap;
}

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

input[type="checkbox"] {
  appearance: none;
  background-color: transparent;
  width: 1.15em;
  height: 1.15em;
  border: 2px solid currentColor;
  border-radius: 0.15em;
  margin-top: 1px;
}

input[type="checkbox"]:checked {
  background-color: currentColor;
  background-image: url("data:image/svg+xml;utf8,<svg width='13' height='10' viewBox='0 0 13 10' fill='none' xmlns='http://www.w3.org/2000/svg'><path d='M5.38587 7.2802L11.9718 0.693573L12.9856 1.70668L5.38587 9.30641L0.826172 4.74671L1.83928 3.73361L5.38587 7.2802Z' fill='white'/></svg>");
  background-repeat: no-repeat;
  background-position: center;
}

input[type="checkbox"]:checked:hover {
  box-shadow: #607b968b 0px 0px 0px 2px;
}

input[type="checkbox"]:not(:checked) {
  border-color: currentColor;
}

input[type="checkbox"]:hover {
  cursor: pointer;
  background-color: currentColor;
  background-image: url("data:image/svg+xml;utf8,<svg width='13' height='10' viewBox='0 0 13 10' fill='none' xmlns='http://www.w3.org/2000/svg'><path d='M5.38587 7.2802L11.9718 0.693573L12.9856 1.70668L5.38587 9.30641L0.826172 4.74671L1.83928 3.73361L5.38587 7.2802Z' fill='white'/></svg>");
  background-repeat: no-repeat;
  background-position: center;
  box-shadow: #607b968b 0px 0px 0px 2px;
}

input[type="checkbox"]:hover:not(:checked) {
  cursor: pointer;
  background-color: rgba(0, 0, 0, 0.1);
  background-image: none;
  box-shadow: #607b968b 0px 0px 0px 2px;
}

input[type="checkbox"]:focus {
  box-shadow: none;
}

@media (max-width: 768px) {
  #blog-posts-case {
    padding: 0px 25px 40px;
  }
}

@media (min-width: 768px) {
  #blog-posts-case {
    grid-template-columns: repeat(2, minmax(0, 1fr));
    padding: 50px 50px 40px;
  }
}

@media (min-width: 1350px) {
  #blog-posts-case {
    grid-template-columns: repeat(3, minmax(0, 1fr));
    padding: 50px 80px 40px;
  }
}

@media (min-width: 1024px) {
  #mobile-page-title {
    display: none;
  }
}
</style>