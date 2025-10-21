<template>
  <div class="code-container font-roboto_mono_regular text-menu-text">
    <!-- text -->
    <div class="text-container pl-0 lg:pl-32">
      <p v-html="text"></p>
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
.code-container {
  display: block;
}

.text-container {
  width: 100%;
  word-wrap: break-word;
}
</style>