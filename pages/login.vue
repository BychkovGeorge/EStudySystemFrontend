<template>
  <main class="col-lg-12 m-0 mw-100 mt-5">
    <div class="row justify-content-center">
      <div class="form-container">
        <b-form-input v-model="mail" type="text" class="mb-3" placeholder="Почта"></b-form-input>
        <b-form-input v-model="password" type="password" class="mb-3" placeholder="Пароль"></b-form-input>
        <b-button @click.prevent="validate" variant="success">Войти</b-button>
      </div>
    </div>
  </main>
</template>

<script>
import axios from 'axios'
import sha256 from 'js-sha256'

export default {
  data () {
    return {
      mail: '',
      password: '',
      jwt: null
    }
  },

  methods: {
    validate () {
      axios.post(`${process.env.backendUrl}/auth/local`, {
        identifier: this.mail,
        password: sha256(this.password)
      }).then((res) => {
        this.jwt = res.data.jwt
        this.$nuxt.$cookiz.set('sessionId', this.jwt)
      })
    }
  }
}
</script>

<style>
.form-container {
  border: 1px solid limegreen;
  border-radius: 6px;
  padding: 10px;
}
</style>
