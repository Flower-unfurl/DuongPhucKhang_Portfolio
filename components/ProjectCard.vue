<template>
    <div id="project" :key="key" class="lg:mx-5">

        <span class="flex text-sm my-3">
            <h3 v-if="index == null" class="text-purplefy font-roboto_mono_bold mr-3">Project {{ key + 1 }}</h3>
            <h3 v-else class="text-purplefy font-roboto_mono_bold mr-3">Project{{ index }}</h3>
            <h4 class="font-roboto_mono_regular text-menu-text"> // {{ project.title }}</h4>
        </span>

    <div 
      id="project-card" 
      class="flex flex-col"
      @click="toggleExpand"
      :class="{ expanded }"
    >
            <div id="window">
                <div class="absolute flex right-3 top-3">
                <img v-for="tech in project.tech" :key="tech" :src="'/icons/techs/filled/' + tech + '.svg'" alt="" class="w-6 h-6 mx-1 hover:opacity-75">
                </div>
                <img id="showcase" :src="project.img" alt="" class="">
            </div>

            <div class="pb-8 pt-6 px-6 border-top">
                <p :class="['text-menu-text font-roboto_mono_regular text-sm mb-5', { description: !expanded }]">
                  {{ expanded ? fullDescription : truncatedDescription }}
                </p>
                <a id="view-button" :href="project.url" target="_blank" @click.stop class="text-white font-roboto_mono_regular py-2 px-4 w-fit text-xs rounded-lg">
                    view-project
                </a>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, computed } from 'vue'
const { project, key, index } = defineProps(['project', 'key', 'index'])

// Truncate description and append "... xem thêm" when too long
const truncatedDescription = computed(() => {
  const desc = project && project.description ? String(project.description) : ''
  const limit = 160
  if (desc.length > limit) {
    return desc.slice(0, limit).trim() + '... xem thêm'
  }
  return desc
})

// Full description fallback
const fullDescription = computed(() => (project && project.description ? String(project.description) : ''))

// Expand/collapse state
const expanded = ref(false)
const toggleExpand = () => {
  expanded.value = !expanded.value
}
</script>

<style scoped>
#project {
  min-width: 400px;
  margin-bottom: 5px;
}

#project-card {
  border: 1px solid #1E2D3D;
  background-color: #011221;
  border-radius: 15px;
  max-width: 400px;
  /* Hover lift effect */
  transform: translateY(0);
  transition: transform 200ms ease, box-shadow 200ms ease, border-color 200ms ease;
  cursor: pointer;
}

#project-card:hover,
#project-card:focus-within {
  transform: translateY(-6px);
  box-shadow: 0 12px 24px rgba(0, 0, 0, 0.25), 0 4px 12px rgba(96, 123, 150, 0.2);
  border-color: #263B50;
}

#window {
  max-height: 120px;
  position: relative;
  overflow: hidden;
  border-top-right-radius: 15px;
  border-top-left-radius: 15px;
}

/* Ensure tech icons overlay the image even when image is transformed */
#window .absolute {
  z-index: 2;
}

#showcase {
  border-top-right-radius: 15px;
  border-top-left-radius: 15px;
  width: 100%;
  object-fit: cover;
  position: relative;
  z-index: 1;
  transition: transform 300ms ease;
}

#project-card:hover #showcase,
#project-card:focus-within #showcase {
  transform: scale(1.03);
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
  #project {
    min-width: 100%;
  }
}

@media (min-width: 768px) {
  #project {
    width: 100%;
    min-width: 100%;
    padding-inline: 5px;
  }
}

@media (min-width: 1350px) {
  #project {
    width: 100%;
    min-width: 100%;
    padding-inline: 20px;
  }
}

</style>