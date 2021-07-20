<template>
  <main v-if="user" class="pb-5 col-lg-12" style="min-height: 100vh">
    <Header class="pb-5" />
    <div class="input-form m-auto pb-5 container row p-5">
      <div v-for="group in user.groups" :key="group.id" class="w-100 mb-5">
        <div class="row m-0 w-100 mb-5 justify-content-center">
          <h3 style="color: white">
            Расписание группы {{ group.name }}
          </h3>
        </div>
        <div v-for="timetable in timetables" :key="timetable.id">
          <div v-if="timetable.timetable">
            <a v-if="timetable.group.id === group.id" style="color: white" :href="timetable.timetable.url" download>{{ timetable.name }}</a>
          </div>
        </div>
      </div>
      <div v-if="user.type === 'teacher'" class="row w-100 ml-0">
        <div v-for="group in user.groups" :key="group.id" class="w-100 mb-5">
          <div class="row m-0 w-100 mb-5 justify-content-center">
            <h3 style="color: white">
              Загрузить расписание группе {{ group.name }}
            </h3>
          </div>
          <div class="row m-0 w-100 mb-5 justify-content-start">
            <form @submit.prevent="handleSubmit(group.id)">
              <div class="mb-3">
                <div class="row w-100 mb-5">
                  <p style="color: white" class="col m-auto">Название расписания:</p>
                  <b-form-input :id="`name${group.id}`" class="col" type="text" placeholder="Название"></b-form-input>
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
      timetables: [],
      file: null,
      loadedFile: null
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
            axios.get(`${process.env.backendUrl}/timetables?group.id=${group.id}`, {
              headers: {
                Authorization: `Bearer ${this.jwt}`
              }
            }).then((timetables) => {
              this.timetables = this.timetables.concat(timetables.data)
            })
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

    handleSubmit (id) {
      if (document.getElementById(`name${id}`).value === '') {
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
            let group = null
            this.user.groups.forEach((el) => {
              if (el.id === id) {
                group = el
              }
            })
            axios.post(`${process.env.backendUrl}/timetables`, {
              timetable: this.loadedFile,
              name: document.getElementById(`name${id}`).value,
              group
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
