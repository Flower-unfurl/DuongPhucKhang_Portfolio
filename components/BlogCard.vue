<template>
  <NuxtLink :to="`/blog/${post._path?.split('/').pop()}`" class="block group">
    <article class="border border-[#1E2D3D] bg-[#011221] rounded-xl overflow-hidden hover:border-[#263B50] transition">
      <div class="relative">
        <img :src="post.cover || '/images/projects/ui-animations2.png'" :alt="post.title" class="w-full aspect-[16/9] object-cover" loading="lazy" />
      </div>
      <div class="p-5">
        <h3 class="text-white font-roboto_mono_bold text-lg group-hover:text-purplefy">{{ post.title }}</h3>
        <p class="text-menu-text font-roboto_mono_regular text-sm mt-2 line-clamp-3">{{ post.description }}</p>
        <div class="flex flex-wrap gap-2 mt-4">
          <span v-for="tag in post.tags || []" :key="tag" class="text-xs text-menu-text border border-[#1E2D3D] rounded-md px-2 py-0.5">#{{ tag }}</span>
        </div>
      </div>
    </article>
  </NuxtLink>
</template>

<script setup lang="ts">
interface BlogPostCard {
  _path?: string
  title: string
  description?: string
  cover?: string
  tags?: string[]
}

const props = defineProps<{ post: BlogPostCard }>()
</script>
