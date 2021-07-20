<template>
  <main v-cloak class="position-relative col-lg-12" style="min-height: 100vh">
    <div class="position-absolute input-form center row p-5">
      <div class="row m-0 w-100 mb-5 justify-content-center">
        <h3 style="color: white">Вход</h3>
      </div>
      <b-form-input v-model="login" class="mb-2" type="text" placeholder="логин" />
      <b-form-input v-model="password" class="mb-2" type="password" placeholder="пароль" />
      <button @click="auth" class="btn btn-success">Войти</button>
    </div>
  </main>
</template>

<script>
import axios from 'axios'

export default {
  data () {
    return {
      login: '',
      password: ''
    }
  },

  methods: {
    auth () {
      const response = axios.post(`${process.env.backendUrl}/auth/local`, {
        identifier: this.login,
        password: this.password
      })
      response.then((onFulfilled) => {
        response.then((dataObject) => {
          this.$nuxt.$cookiz.set('sessionId', dataObject.data.jwt)
          window.location.href = '/main'
        })
      }, (onRejected) => {
        alert('Неверный логин или пароль')
      })
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
