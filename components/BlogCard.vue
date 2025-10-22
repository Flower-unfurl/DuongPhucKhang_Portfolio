<template>
  <NuxtLink :to="post._path || `/blog/${post.slug}`" class="block group">
    <article class="border border-[#1E2D3D] bg-[#011221] rounded-xl overflow-hidden hover:border-[#607B96] transition duration-300">
      <!-- Cover image -->
      <div class="relative aspect-[16/9] overflow-hidden bg-[#1E2D3D]">
        <img 
          :src="post.cover || '/images/projects/ui-animations2.png'" 
          :alt="post.title" 
          class="w-full h-full object-cover group-hover:scale-105 transition duration-300" 
          loading="lazy" 
        />
      </div>
      
      <!-- Card content -->
      <div class="p-5">
        <h3 class="text-white font-roboto_mono_bold text-lg group-hover:text-purplefy transition">
          {{ post.title }}
        </h3>
        
        <p class="text-menu-text font-roboto_mono_regular text-sm mt-2 line-clamp-3 leading-relaxed">
          {{ post.description || 'Nhấp để đọc thêm...' }}
        </p>
        
        <!-- Meta info -->
        <div class="flex items-center gap-3 mt-3 text-xs text-menu-text font-roboto_mono_regular">
          <span v-if="post.date">📅 {{ formatDate(post.date) }}</span>
        </div>
        
        <!-- Tags -->
        <div class="flex flex-wrap gap-2 mt-4">
          <span 
            v-for="tag in (post.tags || [])" 
            :key="tag" 
            class="text-xs text-menu-text border border-[#1E2D3D] rounded-md px-2 py-1 hover:border-[#607B96] hover:text-white transition"
          >
            #{{ tag }}
          </span>
        </div>
      </div>
    </article>
  </NuxtLink>
</template>

<script>
export default {
  props: {
    post: {
      type: Object,
      required: true
    }
  },
  
  methods: {
    formatDate(dateString) {
      if (!dateString) return '';
      
      try {
        const date = new Date(dateString);
        return date.toLocaleDateString('vi-VN', {
          year: 'numeric',
          month: 'short',
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
/* Ensure line-clamp works */
.line-clamp-3 {
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
</style>
