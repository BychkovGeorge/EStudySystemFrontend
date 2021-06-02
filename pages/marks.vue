<template>
  <main v-if="user" class="position-relative col-lg-12" style="min-height: 100vh">
    <Header />
    <div class="position-absolute input-form center row p-5">
      <div class="w-100 mb-5" v-for="group in user.groups" :key="group.id">
        <div class="row m-0 w-100 mb-5 justify-content-center">
          <h3 style="color: white">Оценки группы {{ group.name }}</h3>
        </div>
        <div v-for="mark in marks" :key="mark.id">
          <a style="color: white" :href="mark.markList.url" v-if="mark.group.id === group.id" download>{{ mark.name }}</a>
        </div>
      </div>
    </div>
  </main>
</template>

<script>
import axios from 'axios'

export default {
  data () {
    return {
      jwt: null,
      user: null,
      marks: []
    }
  },

  mounted () {
    if (this.$nuxt.$cookiz.get('sessionId')) {
      this.jwt = this.$nuxt.$cookiz.get('sessionId')
      axios.get(`${process.env.backendUrl}/users/me`, {
        headers: {
          Authorization: `Bearer ${this.jwt}`
        }
      }).then((res) => {
        axios.get(`${process.env.backendUrl}/users/${res.data.id}`, {
          headers: {
            Authorization: `Bearer ${this.jwt}`
          }
        }).then((user) => {
          this.user = user.data
          this.user.groups.forEach((group) => {
            axios.get(`${process.env.backendUrl}/marks?group.id=${group.id}`, {
              headers: {
                Authorization: `Bearer ${this.jwt}`
              }
            }).then((marks) => {
              this.marks = this.marks.concat(marks.data)
            })
          })
        })
      })
    } else {
      alert('Вы не авторизованы')
      window.location.href = '/'
    }
  }
}
</script>

<style>

</style>
