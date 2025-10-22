<template>
  <div class="code-container font-roboto_mono_regular text-menu-text">
    <!-- text -->
    <div class="text-container pl-0 lg:pl-32">
      <div class="ct-markup" v-html="text"></div>
    </div>
  </div>
</template>

<script>

export default {
  props: {
    text: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      animeLib: null,
      isMobile: false,
    }
  },
  watch: {
    text() {
      // New content replaced via v-html, clear processed flag so we rebuild structure
      const container = this.$el ? this.$el.querySelector('.text-container') : null
      if (container && container.dataset) {
        delete container.dataset.processed
      }
      this.$nextTick(() => this.processSections())
    }
  },
  mounted() {
    // Detect mobile
    if (typeof window !== 'undefined') {
      this.isMobile = window.innerWidth < 1024
      window.addEventListener('resize', this.handleResize)
    }
    // Load anime.js on client only and normalize export shape
    if (typeof window !== 'undefined') {
      import('animejs')
        .then(mod => {
          this.animeLib = mod?.default || mod?.anime || mod
          console.log('✅ Anime.js loaded:', !!this.animeLib)
        })
        .catch(err => {
          console.error('❌ Failed to load anime.js:', err)
        })
        .finally(() => {
          // Only process after anime is loaded (or failed)
          this.$nextTick(() => {
            console.log('🔄 Processing sections, mobile:', this.isMobile)
            this.processSections()
          })
        })
    }
  },
  beforeUnmount() {
    if (typeof window !== 'undefined') {
      window.removeEventListener('resize', this.handleResize)
    }
  },
  methods: {
    handleResize() {
      this.isMobile = window.innerWidth < 1024
    },
    processSections() {
      const container = this.$el.querySelector('.text-container')
      if (!container) {
        console.warn('⚠️ No .text-container found')
        return
      }
      // If already processed and structure exists, just re-run animations
      const alreadyProcessed = container.dataset.processed === 'true'
      const hasSections = container.querySelectorAll('.ct-section').length > 0
      if (alreadyProcessed && hasSections) {
        console.log('♻️ Re-running animations on already processed content')
        this.runAnimations(container)
        return
      }
      const headings = container.querySelectorAll('h1, h2, h3')
      console.log(`📝 Found ${headings.length} headings`)
      if (!headings.length) return
      headings.forEach(h => h.classList.add('ct-heading'))
      headings.forEach((h, idx) => {
        // Ensure we work relative to the heading's actual parent, not just container
        const parent = h.parentNode
        if (!parent) return

        // Capture the sibling that comes after the heading BEFORE moving it
        let cursor = h.nextSibling

        const section = document.createElement('div')
        section.className = 'ct-section'
        // Insert the new section before the heading within its own parent
        parent.insertBefore(section, h)

        // Desktop: Wrap heading in layers for stripe + grid effects
        if (!this.isMobile) {
          console.log(`🖥️ Desktop mode: Creating wrapper for heading ${idx + 1}`)
          const headingWrapper = document.createElement('div')
          headingWrapper.className = 'ct-heading-wrapper'

          // Grid background (parallax effect)
          const grid = document.createElement('div')
          grid.className = 'ct-grid'
          headingWrapper.appendChild(grid)

          // Stripe pattern overlay
          const stripes = document.createElement('div')
          stripes.className = 'ct-stripes'
          headingWrapper.appendChild(stripes)

          // Heading itself (moving node from parent to wrapper)
          headingWrapper.appendChild(h)
          section.appendChild(headingWrapper)
        } else {
          console.log(`📱 Mobile mode: Splitting letters for heading ${idx + 1}`)
          // Mobile: Split text into letters for stagger effect
          const text = h.textContent || ''
          h.innerHTML = ''
          text.split('').forEach((char) => {
            const span = document.createElement('span')
            span.className = 'ct-char'
            span.textContent = char === ' ' ? '\u00A0' : char // preserve spaces
            h.appendChild(span)
          })
          section.appendChild(h)
        }

        const divider = document.createElement('div')
        divider.className = 'ct-divider'
        section.appendChild(divider)

        const content = document.createElement('div')
        content.className = 'ct-content'
        // Move subsequent siblings from the ORIGINAL parent into this content block
        while (
          cursor &&
          cursor.parentNode === parent &&
          !(cursor.tagName && /^H[1-3]$/.test(cursor.tagName))
        ) {
          const next = cursor.nextSibling
          content.appendChild(cursor)
          cursor = next
        }
        section.appendChild(content)
      })
      container.dataset.processed = 'true'
      console.log('✨ Starting animations...')
      this.runAnimations(container)
    },
    runAnimations(container) {
      if (!this.animeLib) return
      
      if (!this.isMobile) {
        // Desktop animations: stripe + grid + underline + shine
        
        // 1. Grid parallax (subtle movement)
        this.animeLib({
          targets: container.querySelectorAll('.ct-grid'),
          translateX: [-10, 0],
          translateY: [-5, 0],
          opacity: [0, 0.15],
          duration: 800,
          easing: 'easeOutQuad',
          delay: (_, i) => i * 120
        })
        
        // 2. Stripe pattern reveal (clip-path or scaleX)
        this.animeLib({
          targets: container.querySelectorAll('.ct-stripes'),
          scaleX: [0, 1],
          opacity: [0, 0.6],
          duration: 700,
          easing: 'easeOutExpo',
          delay: (_, i) => 50 + i * 120,
          transformOrigin: 'left center'
        })
        
        // 3. Heading text fade + scale
        this.animeLib({
          targets: container.querySelectorAll('.ct-heading'),
          opacity: [0, 1],
          scaleX: [0.95, 1],
          duration: 600,
          easing: 'easeOutQuad',
          delay: (_, i) => 100 + i * 120,
          transformOrigin: 'left center'
        })
        
        // 4. Divider sweep (underline)
        this.animeLib({
          targets: container.querySelectorAll('.ct-divider'),
          scaleX: [0, 1],
          duration: 750,
          easing: 'easeOutExpo',
          delay: (_, i) => 200 + i * 120,
          transformOrigin: 'left center'
        })
        
        // 5. Content stagger (paragraphs)
        const contentElements = container.querySelectorAll('.ct-content')
        contentElements.forEach((el, idx) => {
          const children = Array.from(el.children).filter(c => c.tagName === 'BR' ? false : true)
          this.animeLib({
            targets: children.length > 0 ? children : [el],
            opacity: [0, 1],
            translateY: [8, 0],
            duration: 550,
            easing: 'easeOutQuad',
            delay: this.animeLib.stagger(60, { start: 350 + idx * 140 })
          })
        })
        
      } else {
        // Mobile animations: letter-by-letter + simple fade for content
        
        // 1. Letter-by-letter stagger
        this.animeLib({
          targets: container.querySelectorAll('.ct-char'),
          opacity: [0, 1],
          translateY: [8, 0],
          duration: 400,
          easing: 'easeOutQuad',
          delay: this.animeLib.stagger(25) // 25ms per letter
        })
        
        // 2. Divider simple reveal
        this.animeLib({
          targets: container.querySelectorAll('.ct-divider'),
          scaleX: [0, 1],
          duration: 500,
          easing: 'easeOutQuad',
          delay: (_, i) => 300 + i * 100,
          transformOrigin: 'left center'
        })
        
        // 3. Content simple fade-in
        this.animeLib({
          targets: container.querySelectorAll('.ct-content'),
          opacity: [0, 1],
          duration: 450,
          easing: 'easeOutQuad',
          delay: (_, i) => 400 + i * 100
        })
      }
    }
  }
};
</script>

<style>
/* Remove 'scoped' to allow CSS to apply to dynamically created elements */
/* Use specific selectors to avoid global conflicts */
.code-container {
  display: block;
}

.code-container .text-container {
  width: 100%;
  text-align: justify;
  word-wrap: break-word;
}

/* ========== HEADING WRAPPER (DESKTOP) ========== */
.code-container .ct-heading-wrapper {
  position: relative;
  margin-top: 24px;
  margin-bottom: 8px;
  overflow: visible; /* allow effects to be visible */
  min-height: 32px;
}

/* Grid background (parallax subtle dots) */
.code-container .ct-grid {
  position: absolute;
  top: -10px;
  left: -10px;
  right: -10px;
  bottom: -10px;
  background-image: 
    radial-gradient(circle, rgba(96, 123, 150, 0.2) 1px, transparent 1px);
  background-size: 20px 20px;
  pointer-events: none;
  z-index: 0;
  opacity: 0;
}

/* Stripe pattern overlay */
.code-container .ct-stripes {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: repeating-linear-gradient(
    45deg,
    rgba(67, 217, 173, 0.08),
    rgba(67, 217, 173, 0.08) 8px,
    rgba(121, 159, 251, 0.06) 8px,
    rgba(121, 159, 251, 0.06) 16px
  );
  pointer-events: none;
  z-index: 1;
  transform: scaleX(0);
  transform-origin: left center;
  opacity: 0;
}

/* Title size and emphasis for h1 (# in markdown) */
.code-container .text-container h1.ct-heading {
  position: relative;
  z-index: 2;
  font-size: 20px !important; /* override global h2/h3 */
  font-weight: 700 !important;
  margin: 0 !important;
  padding: 4px 0;
  text-align: left;
  /* Shine effect via background-clip */
  background: linear-gradient(90deg, #fff 0%, #a5c5e5 50%, #fff 100%) !important;
  background-size: 200% 100% !important;
  background-clip: text !important;
  -webkit-background-clip: text !important;
  -webkit-text-fill-color: transparent !important;
  animation: ct-shine 3s ease-in-out infinite;
}

@keyframes ct-shine {
  0%, 100% { background-position: 200% center; }
  50% { background-position: -100% center; }
}

/* Optional: similar styling for h2/h3 */
.code-container .text-container h2.ct-heading,
.code-container .text-container h3.ct-heading {
  position: relative;
  z-index: 2;
  font-size: 18px !important;
  font-weight: 700 !important;
  margin: 0 !important;
  padding: 4px 0;
  text-align: left;
  background: linear-gradient(90deg, #fff 0%, #a5c5e5 50%, #fff 100%) !important;
  background-size: 200% 100% !important;
  background-clip: text !important;
  -webkit-background-clip: text !important;
  -webkit-text-fill-color: transparent !important;
  animation: ct-shine 3s ease-in-out infinite;
}

/* ========== MOBILE: LETTER-BY-LETTER ========== */
.code-container .ct-char {
  display: inline-block;
  opacity: 0;
  transform: translateY(8px);
}

/* ========== DIVIDER (UNDERLINE) ========== */
.code-container .ct-divider {
  width: 100%;
  height: 2px;
  background: linear-gradient(90deg, #43D9AD 0%, #799ffb 100%);
  transform: scaleX(0);
  margin: 6px 0 10px;
}

/* ========== CONTENT ========== */
.code-container .ct-content {
  margin-bottom: 16px;
}

/* On mobile, simplify heading (no wrapper, no shine to reduce weight) */
@media (max-width: 1023px) {
  .code-container .text-container h1.ct-heading,
  .code-container .text-container h2.ct-heading,
  .code-container .text-container h3.ct-heading {
    background: none !important;
    color: white !important;
    animation: none !important;
    -webkit-text-fill-color: white !important;
    -webkit-background-clip: unset !important;
    background-clip: unset !important;
  }
  
  .code-container .ct-heading-wrapper {
    margin-top: 20px;
    margin-bottom: 6px;
  }
}
</style>