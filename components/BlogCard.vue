<template>
  <div id="blog-post" :key="key" class="lg:mx-5">
    <span class="flex text-sm my-3">
      <h3 v-if="index == null" class="text-purplefy font-roboto_mono_bold mr-3">Post {{ key + 1 }}</h3>
      <h3 v-else class="text-purplefy font-roboto_mono_bold mr-3">Post{{ index + 1 }}</h3>
      <h4 class="font-roboto_mono_regular text-menu-text"> // {{ post.title1 }}</h4>
    </span>

    <div id="blog-card" class="flex flex-col">
      <div id="window">
        <div class="absolute flex right-3 top-3">
          <img 
            v-for="tag in (post.tags || [])" 
            :key="tag" 
            alt="" 
            class="w-6 h-6 mx-1 hover:opacity-75"
            :title="tag"
          >
        </div>
        <img 
          id="showcase" 
          :src="post.cover || '/images/projects/ui-animations2.png'" 
          :alt="post.title"
          loading="lazy"
        >
      </div>

      <div class="pb-8 pt-6 px-6 border-top">
        <p class="text-menu-text font-roboto_mono_regular text-sm mb-5">
          {{ post.description || 'Nhấp để đọc thêm...' }}
        </p>
        <div class="flex items-center justify-between">
          <NuxtLink 
            :to="post._path || `/blog/${post.slug}`"
            id="view-button" 
            class="text-white font-roboto_mono_regular py-2 px-4 w-fit text-xs rounded-lg"
          >
            view-post
          </NuxtLink>
          <span v-if="post.date" class="text-menu-text font-roboto_mono_regular text-xs">
            📅 {{ formatDate(post.date) }}
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    post: {
      type: Object,
      required: true
    },
    key: {
      type: Number,
      default: null
    },
    index: {
      type: Number,
      default: null
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
#blog-post {
  min-width: 400px;
  margin-bottom: 5px;
}

#blog-card {
  border: 1px solid #1E2D3D;
  background-color: #011221;
  border-radius: 15px;
  max-width: 400px;
}

#window {
  max-height: 145px;
  position: relative;
  overflow: hidden;
}

#showcase {
  border-top-right-radius: 15px;
  border-top-left-radius: 15px;
  width: 100%;
  object-fit: cover;
}

#view-button {
  background-color: #1C2B3A;
}

#view-button:hover {
  background-color: #263B50;
}

@media (max-width: 768px) {
  #blog-post {
    min-width: 100%;
  }
}

@media (min-width: 768px) {
  #blog-post {
    width: 100%;
    min-width: 100%;
    padding-inline: 5px;
  }
}

@media (min-width: 1350px) {
  #blog-post {
    width: 100%;
    min-width: 100%;
    padding-inline: 20px;
  }
}
</style>
