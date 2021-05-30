<template>
  <main v-if="user" class="position-relative col-lg-12" style="min-height: 100vh">
    <div class="position-absolute input-form center row p-5">
      <div class="row m-0 w-100 mb-5 justify-content-center">
        <h3 style="color: white">Ваши данные</h3>
      </div>
      <div class="row w-100 justify-content-start mb-2">
        <h4 v-if="user.type === 'student'" style="color: white">Студент группы: {{user.groups[0].name}}</h4>
        <h4 v-if="user.type === 'teacher'" style="color: white">Преподаватель</h4>
      </div>
      <div class="row w-100 justify-content-start">
        <ul style="list-style: none">
          <li style="color: white">{{ user.name }}</li>
          <li style="color: white">{{ user.middleName }}</li>
          <li style="color: white">{{ user.surname }}</li>
        </ul>
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
      user: null
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
.center {
  top: 50%;
  left: 50%;
  margin-right: -50%;
  transform: translate(-50%, -50%);
}

.input-form {
  border: 1px solid #fff;
}

.input-form:hover {
  border: 1px solid #dfff;
  box-shadow: 0 5px 0 rgba(255, 255, 255, .3);
}
</style>
