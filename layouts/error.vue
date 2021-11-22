<template>
  <div>
    <p v-if="error.statusCode === 404">Page not found</p>
    <NuxtLink to="/">點此回首頁</NuxtLink>
  </div>
</template>
<script>
export default {
  props: ['error'],
}
</script>