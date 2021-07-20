<template>
  <main v-if="user" class="pb-5 col-lg-12" style="min-height: 100vh">
    <Header class="pb-5" />
    <div class="input-form m-auto pb-5 container row p-5">
      <div class="w-100 mb-5">
        <div class="row m-0 w-100 mb-5 justify-content-center">
          <h3 style="color: white">
            Портфолио {{ user.name }}&nbsp;{{ user.surname }}
          </h3>
        </div>
        <div v-for="portfolio in portfolios" :key="portfolio.id">
          <div v-if="portfolio.file">
            <a style="color: white" :href="portfolio.file.url" download>{{ portfolio.name }}</a>
          </div>
        </div>
      </div>
      <div class="row w-100 ml-0">
        <div class="w-100 mb-5">
          <div class="row m-0 w-100 mb-5 justify-content-center">
            <h3 style="color: white">
              Загрузить новый документ в портфолио
            </h3>
          </div>
          <div class="row m-0 w-100 mb-5 justify-content-start">
            <form @submit.prevent="handleSubmit()">
              <div class="mb-3">
                <div class="row w-100 mb-5">
                  <p style="color: white" class="col m-auto">Название файла:</p>
                  <b-form-input class="col" type="text" v-model="name" placeholder="Название"></b-form-input>
                </div>
                <label>
                  <input type="file" class="ml-3 download-link" @change="selectFile">
                  <span style="color: white; cursor: pointer" class="download-link pointer choose-file">Прикрепить</span>
                </label>
              </div>
              <div class="mb-2" :class="{'justify-content-center':width < 992}">
                <button class="btn btn-success">
                  Загрузить
                </button>
              </div>
            </form>
          </div>
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
      portfolios: [],
      file: null,
      loadedFile: null,
      name: ''
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
          axios.get(`${process.env.backendUrl}/portfolios?users_permissions_user.id=${this.user.id}`, {
            headers: {
              Authorization: `Bearer ${this.jwt}`
            }
          }).then((portfolios) => {
            this.portfolios = this.portfolios.concat(portfolios.data)
          })
        })
      })
    } else {
      alert('Вы не авторизованы')
      window.location.href = '/'
    }
  },

  methods: {
    selectFile (event) {
      this.file = event.target.files[0]
    },

    handleSubmit () {
      if (this.name === '') {
        alert('Введите название')
        return
      }
      const data = new FormData()
      data.append('files', this.file)
      const response = axios.post(`${process.env.backendUrl}/upload`, data)

      response.then((dataObject) => {
        const fileId = dataObject.data[0].id
        const file = axios.get(`${process.env.backendUrl}/upload/files/${fileId}`, {
          headers: {
            Authorization:
              `Bearer ${this.jwt}`
          }
        })
        file.then((onFulfilled) => {
          file.then((dataObject) => {
            this.loadedFile = dataObject.data
            axios.post(`${process.env.backendUrl}/portfolios`, {
              file: this.loadedFile,
              name: this.name,
              users_permissions_user: this.user
            }, {
              headers: {
                Authorization:
                  `Bearer ${this.jwt}`
              }
            }).then((onFulfilled) => {
              alert('Файл загружен')
              window.location.reload()
            })
          })
        })
      })
    }
  }
}
</script>

<style>
label input {
  display: none;
}
</style>
