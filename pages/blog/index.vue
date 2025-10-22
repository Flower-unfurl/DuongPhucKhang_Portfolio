<template>
  <main class="flex flex-col flex-auto overflow-hidden">
    <div id="mobile-page-title">
      <h2>_blog</h2>
    </div>

    <div class="w-full border-bot hidden lg:flex items-center tab-height">
      <p class="font-roboto_mono_regular text-menu-text text-sm px-3">blog</p>
    </div>

    <section class="w-full px-6 lg:px-12 py-6 overflow-y-auto">
      <!-- Controls: Search + Tags -->
      <div class="flex flex-col gap-4 mb-6">
        <div class="flex items-center gap-3 flex-wrap">
          <input 
            v-model="searchQuery" 
            type="search" 
            placeholder="Tìm kiếm bài viết..." 
            class="bg-transparent border border-[#1E2D3D] rounded-lg px-3 py-2 text-sm text-white w-full max-w-xl placeholder-[#465E77] focus:outline-none focus:border-[#607B96]" 
          />
          <select 
            v-model="sortOrder" 
            class="bg-[#011221] border border-[#1E2D3D] rounded-lg px-3 py-2 text-sm text-menu-text focus:outline-none focus:border-[#607B96]"
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

      <!-- Grid -->
      <div v-if="paginatedPosts.length > 0" class="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-3 gap-6">
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
          class="px-4 py-2 rounded-lg border border-[#1E2D3D] text-menu-text disabled:opacity-40 hover:border-[#607B96] transition" 
          :disabled="currentPage === 1" 
          @click="currentPage--"
        >
          ← Trước
        </button>
        <span class="text-menu-text text-sm">
          Trang {{ currentPage }} / {{ totalPages }}
        </span>
        <button 
          class="px-4 py-2 rounded-lg border border-[#1E2D3D] text-menu-text disabled:opacity-40 hover:border-[#607B96] transition" 
          :disabled="currentPage === totalPages" 
          @click="currentPage++"
        >
          Sau →
        </button>
      </div>
    </section>
  </main>
</template>

<script setup lang="ts">
import { computed, ref, watch } from 'vue'

interface BlogPost {
  _path: string
  title: string
  description?: string
  date: string
  tags?: string[]
  cover?: string
  draft?: boolean
}

useHead({ 
  title: 'Blog | Dương Phúc Khang',
  meta: [
    { name: 'description', content: 'Chia sẻ kiến thức và kinh nghiệm phát triển web' }
  ]
})

// Fetch all blog posts using Nuxt Content auto-imported composable
const { data: postsData } = await useAsyncData('blog-posts', async () => {
  const posts = await queryContent<BlogPost>('/blog')
    .where({ draft: { $ne: true } })
    .sort({ date: -1 })
    .find()
  return posts
})

// Reactive state
const searchQuery = ref('')
const selectedTag = ref<string>('all')
const sortOrder = ref<'date-desc' | 'date-asc'>('date-desc')
const currentPage = ref(1)
const pageSize = 9

// Computed: available tags
const availableTags = computed(() => {
  const tagSet = new Set<string>()
  const posts = postsData.value || []
  
  if (Array.isArray(posts)) {
    posts.forEach((post: BlogPost) => {
      if (post.tags && Array.isArray(post.tags)) {
        post.tags.forEach(tag => tagSet.add(tag))
      }
    })
  }
  
  return Array.from(tagSet).sort()
})

// Computed: filtered posts
const filteredPosts = computed(() => {
  let posts = Array.isArray(postsData.value) ? postsData.value : []
  
  // Filter by tag
  if (selectedTag.value !== 'all') {
    posts = posts.filter(post => 
      post.tags?.includes(selectedTag.value)
    )
  }
  
  // Filter by search query
  if (searchQuery.value.trim()) {
    const query = searchQuery.value.toLowerCase()
    posts = posts.filter(post => 
      post.title?.toLowerCase().includes(query) ||
      post.description?.toLowerCase().includes(query)
    )
  }
  
  // Sort
  posts = [...posts].sort((a, b) => {
    const dateA = new Date(a.date).getTime()
    const dateB = new Date(b.date).getTime()
    return sortOrder.value === 'date-desc' ? dateB - dateA : dateA - dateB
  })
  
  return posts
})

// Computed: total pages
const totalPages = computed(() => 
  Math.max(1, Math.ceil(filteredPosts.value.length / pageSize))
)

// Computed: paginated posts
const paginatedPosts = computed(() => {
  const start = (currentPage.value - 1) * pageSize
  return filteredPosts.value.slice(start, start + pageSize)
})

// Reset to page 1 when filters change
watch([searchQuery, selectedTag, sortOrder], () => {
  currentPage.value = 1
})

// Helper: tag button class
function tagButtonClass(tag: string) {
  const isActive = selectedTag.value === tag
  return [
    'text-xs whitespace-nowrap border rounded-md px-3 py-1.5 transition font-roboto_mono_regular',
    isActive 
      ? 'text-white border-[#607B96] bg-[#1C2B3A]' 
      : 'text-menu-text border-[#1E2D3D] hover:border-[#607B96] hover:text-white'
  ]
}
</script>
