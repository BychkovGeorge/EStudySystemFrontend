<template>
  <main v-if="user" class="pb-5 col-lg-12" style="min-height: 100vh">
    <Header class="pb-5" />
    <div v-if="user.type === 'teacher'" class="input-form m-auto pb-5 container row p-5">
      <div class="w-100 mb-5">
        <div class="row m-0 w-100 mb-5 justify-content-center">
          <h3 style="color: white">
            Учебные материалы {{ user.name }}&nbsp;{{ user.surname }}
          </h3>
        </div>
        <div v-for="material in materialsForTeachers" :key="material.id">
          <div v-if="material.file">
            <a style="color: white" :href="material.file.url" download>{{ material.name }}</a>
          </div>
        </div>
      </div>
      <div class="row w-100 ml-0">
        <div class="w-100 mb-5">
          <div class="row m-0 w-100 mb-5 justify-content-center">
            <h3 style="color: white">
              Загрузить новый файл
            </h3>
          </div>
          <div class="row m-0 w-100 mb-5 justify-content-start">
            <form @submit.prevent="handleSubmit()">
              <div class="mb-3">
                <div class="row w-100 mb-5">
                  <p style="color: white" class="col m-auto">Название файла:</p>
                  <b-form-input v-model="name" class="col" type="text" placeholder="Название"></b-form-input>
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

    <div v-else class="input-form m-auto pb-5 container row p-5">
      <div v-for="teacher in materialsForStudents" :key="teacher.user.id" class="w-100">
        <div class="w-100 mb-5">
          <div class="row m-0 w-100 mb-5 justify-content-center">
            <h3 style="color: white">
              Учебные материалы преподавателя {{ teacher.user.name }}&nbsp;{{ teacher.user.surname }}
            </h3>
          </div>
          <div v-for="material in teacher.materials.data" :key="material.id">
            <div v-if="material.file">
              <a style="color: white" :href="material.file.url" download>{{ material.name }}</a>
            </div>
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
      user: null,
      materialsForStudents: [],
      materialsForTeachers: [],
      name: '',
      file: null
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
          if (this.user.type === 'teacher') {
            axios.get(`${process.env.backendUrl}/study-materials?users_permissions_user.id=${this.user.id}`, {
              headers: {
                Authorization: `Bearer ${this.jwt}`
              }
            }).then((materials) => {
              this.materialsForTeachers = materials.data
            })
          } else {
            axios.get(`${process.env.backendUrl}/users`, {
              headers: {
                Authorization: `Bearer ${this.jwt}`
              }
            }).then((allUsers) => {
              allUsers.data.forEach((user) => {
                user.groups.forEach((group) => {
                  if (group.id === this.user.groups[0].id && user.type === 'teacher') {
                    axios.get(`${process.env.backendUrl}/study-materials?users_permissions_user.id=${user.id}`, {
                      headers: {
                        Authorization: `Bearer ${this.jwt}`
                      }
                    }).then((materials) => {
                      const temp = {
                        user,
                        materials
                      }
                      this.materialsForStudents.push(temp)
                    })
                  }
                })
              })
            })
          }
        })
      })
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
            axios.post(`${process.env.backendUrl}/study-materials`, {
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
