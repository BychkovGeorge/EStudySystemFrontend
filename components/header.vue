<template>
  <div class="pt-5 row justify-content-center">
    <div class="header-container py-3">
      <nuxt-link class="mr-5 ml-5" style="color: white" to="/main">Главная</nuxt-link>
      <nuxt-link class="mr-5" style="color: white" to="/marks">Оценки</nuxt-link>
      <nuxt-link class="mr-5" style="color: white" to="/timetable">Расписание</nuxt-link>
      <nuxt-link class="mr-5" style="color: white" to="/tasks">Задания</nuxt-link>
      <nuxt-link class="mr-5" style="color: white" to="/portfolio">Портфолио</nuxt-link>
    </div>
  </div>
</template>

<style scoped>
.header-container {
  border: 1px solid white;
  border-radius: 5px;
  max-width: 50%;
}
</style>
