<template>
  <div id="blog-post" :key="key" class="lg:mx-5">
    <span class="flex text-sm my-3">
      <h3 v-if="index == null" class="text-purplefy font-roboto_mono_bold mr-3">Post {{ key + 1 }}</h3>
      <h3 v-else class="text-purplefy font-roboto_mono_bold mr-3">Post{{ index + 1 }}</h3>
      <h4 class="font-roboto_mono_regular text-menu-text"> // {{ post.title1 }}</h4>
    </span>

    <div 
      id="blog-card" 
      class="flex flex-col"
      @click="toggleExpand"
      :class="{ expanded }"
    >
      <div id="window">
        <div class="absolute flex right-3 top-3"></div>
        <img 
          id="showcase" 
          :src="post.cover || '/images/projects/ui-animations2.png'" 
          :alt="post.title"
          loading="lazy"
        >
      </div>

      <div class="pb-8 pt-6 px-6 border-top">
        <p :class="['text-menu-text font-roboto_mono_regular text-sm mb-5', { description: !expanded }]">
          {{ expanded ? fullDescription : truncatedDescription }}
        </p>
        <div class="flex items-center justify-between">
          <NuxtLink 
            :to="post._path || `/blog/${post.slug}`"
            id="view-button" 
            class="text-white font-roboto_mono_regular py-2 px-4 w-fit text-xs rounded-lg"
            @click.stop
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
  data() {
    return {
      expanded: false
    }
  },
  
  computed: {
    // Truncate description and append "... xem thêm" when it is too long
    truncatedDescription() {
      const desc = (this.post && this.post.description) ? String(this.post.description) : 'Nhấp để đọc thêm...';
      const limit = 160; // character limit as a fallback for browsers without line-clamp
      if (desc.length > limit) {
        return desc.slice(0, limit).trim() + '... xem thêm';
      }
      return desc;
    },
    fullDescription() {
      return (this.post && this.post.description) ? String(this.post.description) : '';
    }
  },
  
  methods: {
    toggleExpand() {
      this.expanded = !this.expanded;
    },
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
  /* Hover lift effect */
  transform: translateY(0);
  transition: transform 200ms ease, box-shadow 200ms ease, border-color 200ms ease;
  cursor: pointer;
}

#blog-card:hover,
#blog-card:focus-within {
  transform: translateY(-6px);
  box-shadow: 0 12px 24px rgba(0, 0, 0, 0.25), 0 4px 12px rgba(96, 123, 150, 0.2);
  border-color: #263B50;
}

#window {
  max-height: 145px;
  position: relative;
  overflow: hidden;
  border-top-right-radius: 15px;
  border-top-left-radius: 15px;
}

#showcase {
  width: 100%;
  object-fit: cover;
  transition: transform 300ms ease;
}

#blog-card:hover #showcase,
#blog-card:focus-within #showcase {
  transform: scale(1.03);
}

#view-button {
  background-color: #1C2B3A;
}

#view-button:hover {
  background-color: #263B50;
}

/* Clamp description to 3 lines for consistent card heights */
.description {
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  line-clamp: 3;
  overflow: hidden;
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
