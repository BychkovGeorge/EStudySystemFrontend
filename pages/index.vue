<template>
  <main/>
</template>

<script>
export default {
  mounted () {
    if (!this.$nuxt.$cookiz.get('sessionId')) {
      document.location.href = '/login'
    }
  }
}
</script>

<style>

</style>
