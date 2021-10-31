<template>
  <div v-if="up == true">
    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" style="fill: rgba(35, 164, 72, 1); transform: ; msfilter: ">
      <path d="m10 10.414 4 4 5.707-5.707L22 11V5h-6l2.293 2.293L14 11.586l-4-4-7.707 7.707 1.414 1.414z"></path>
    </svg>
  </div>
  <div v-else>
    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" style="fill: rgba(164, 51, 35, 1); transform: ; msfilter: ">
      <path d="m14 9.586-4 4-6.293-6.293-1.414 1.414L10 16.414l4-4 4.293 4.293L16 19h6v-6l-2.293 2.293z"></path>
    </svg>
  </div>
</template>

<script>
export default {
  props: {
    up: {
      type: Boolean,
      required: true,
      default: true,
    },
  },
};
</script>

<style scoped>
svg {
  position: relative;
}
</style>
