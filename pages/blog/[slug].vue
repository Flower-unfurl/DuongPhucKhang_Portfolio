<template>
  <main class="flex flex-col flex-auto overflow-hidden">
    <div id="mobile-page-title">
      <h2>_blog</h2>
    </div>

    <!-- Desktop tab -->
    <div class="w-full border-bot hidden lg:flex items-center tab-height">
      <div class="flex items-center border-right h-full px-3">
        <p class="font-roboto_mono_regular text-menu-text text-sm">{{ doc?.title || 'blog' }}</p>
      </div>
    </div>

    <section class="w-full h-full overflow-y-auto">
      <div class="px-6 lg:px-12 py-6 max-w-7xl mx-auto grid grid-cols-1 lg:grid-cols-[1fr_280px] gap-8">
        
        <!-- Main content -->
        <div>
          <div v-if="doc" class="border border-[#1E2D3D] bg-[#011221] rounded-xl overflow-hidden">
            <!-- Cover image -->
            <img 
              v-if="doc.cover" 
              :src="doc.cover" 
              :alt="doc.title" 
              class="w-full aspect-[16/9] object-cover" 
            />
            
            <!-- Article content -->
            <div class="p-6 lg:p-8">
              <h1 class="text-white font-roboto_mono_bold text-2xl lg:text-3xl">
                {{ doc.title }}
              </h1>
              
              <div class="flex flex-wrap items-center gap-3 mt-3 text-sm text-menu-text font-roboto_mono_regular">
                <span>📅 {{ formattedDate }}</span>
                <span v-if="readingTime">• ⏱️ {{ readingTime }} phút đọc</span>
              </div>
              
              <div class="flex flex-wrap gap-2 mt-4">
                <span 
                  v-for="tag in doc.tags || []" 
                  :key="tag" 
                  class="text-xs text-menu-text border border-[#1E2D3D] rounded-md px-2 py-1 hover:border-[#607B96] hover:text-white transition"
                >
                  #{{ tag }}
                </span>
              </div>
              
              <!-- Blog post body -->
              <article class="mt-8 blog-content">
                <ContentRenderer :value="doc" />
              </article>
            </div>
          </div>
          
          <!-- Loading state -->
          <div v-else class="flex items-center justify-center py-20 text-menu-text">
            <p>Đang tải...</p>
          </div>
        </div>

        <!-- Table of contents (desktop only) -->
        <aside class="hidden lg:block">
          <div class="sticky top-6 border border-[#1E2D3D] rounded-xl p-4 bg-[#011221]">
            <h3 class="text-sm text-white font-roboto_mono_bold mb-3">📑 Mục lục</h3>
            <nav v-if="tocLinks.length > 0" class="text-xs text-menu-text space-y-2">
              <a 
                v-for="link in tocLinks" 
                :key="link.id" 
                :href="`#${link.id}`" 
                class="block hover:text-white transition pl-2 border-l-2 border-transparent hover:border-purplefy font-roboto_mono_regular"
              >
                {{ link.text }}
              </a>
            </nav>
            <p v-else class="text-xs text-menu-text">Không có mục lục</p>
          </div>
        </aside>
      </div>
    </section>
  </main>
</template>

<script setup lang="ts">
import { computed } from 'vue'

interface TocLink {
  id: string
  text: string
  depth: number
}

interface BlogPost {
  title: string
  description?: string
  date: string
  tags?: string[]
  cover?: string
  body?: {
    toc?: {
      links?: TocLink[]
    }
  }
}

const route = useRoute()
const slug = route.params.slug as string

useHead({
  title: computed(() => doc.value ? `${doc.value.title} | Blog` : 'Blog'),
  meta: computed(() => [
    { name: 'description', content: doc.value?.description || 'Blog post' }
  ])
})

// Fetch blog post
const { data: doc } = await useAsyncData(`blog-${slug}`, async () => {
  try {
    const post = await queryContent<BlogPost>(`/blog/${slug}`).findOne()
    return post
  } catch (error) {
    console.error('Error loading blog post:', error)
    return null
  }
})

// Computed: formatted date
const formattedDate = computed(() => {
  if (!doc.value?.date) return ''
  const date = new Date(doc.value.date)
  return date.toLocaleDateString('vi-VN', {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  })
})

// Computed: table of contents links
const tocLinks = computed(() => {
  return doc.value?.body?.toc?.links || []
})

// Computed: reading time (simple estimation: 200 words per minute)
const readingTime = computed(() => {
  if (!doc.value) return null
  // Simple word count from body text
  const text = JSON.stringify(doc.value.body || '')
  const wordCount = text.split(/\s+/).length
  return Math.ceil(wordCount / 200)
})
</script>

<style scoped>
/* Blog content styling */
:deep(.blog-content) {
  @apply text-menu-text font-roboto_mono_regular leading-relaxed;
}

:deep(.blog-content h1),
:deep(.blog-content h2),
:deep(.blog-content h3),
:deep(.blog-content h4) {
  @apply text-white font-roboto_mono_bold mt-6 mb-3;
  scroll-margin-top: 2rem;
}

:deep(.blog-content h1) {
  @apply text-2xl;
}

:deep(.blog-content h2) {
  @apply text-xl border-b border-[#1E2D3D] pb-2;
}

:deep(.blog-content h3) {
  @apply text-lg;
}

:deep(.blog-content p) {
  @apply my-4;
}

:deep(.blog-content a) {
  @apply text-purplefy hover:underline;
}

:deep(.blog-content ul),
:deep(.blog-content ol) {
  @apply my-4 ml-6;
}

:deep(.blog-content li) {
  @apply my-2;
}

:deep(.blog-content ul li) {
  list-style-type: disc;
}

:deep(.blog-content ol li) {
  list-style-type: decimal;
}

:deep(.blog-content code) {
  @apply bg-[#1E2D3D] text-greenfy px-1.5 py-0.5 rounded text-sm;
}

:deep(.blog-content pre) {
  @apply bg-[#1E2D3D] rounded-lg p-4 overflow-x-auto my-4;
}

:deep(.blog-content pre code) {
  @apply bg-transparent p-0;
}

:deep(.blog-content blockquote) {
  @apply border-l-4 border-purplefy pl-4 italic my-4;
}

:deep(.blog-content img) {
  @apply rounded-lg my-4;
}

:deep(.blog-content table) {
  @apply w-full my-4 border border-[#1E2D3D];
}

:deep(.blog-content th),
:deep(.blog-content td) {
  @apply border border-[#1E2D3D] px-3 py-2;
}

:deep(.blog-content th) {
  @apply bg-[#1E2D3D] text-white font-roboto_mono_bold;
}
</style>
