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