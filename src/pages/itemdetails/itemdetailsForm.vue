<template>
  <q-dialog v-model="dialog" persistent>
    <q-card style="width: 600px; max-width: 60vw">
      <q-card-section class="row items-center q-pb-none">
        <div class="text-h6">{{ name }}</div>
        <q-space />
        <q-btn icon="close" flat round dense @click="close()" />
      </q-card-section>

      <q-card-section>
        <q-form @submit="submit" class="q-gutter-md">
          <div class="q-pa-sm">
            <div class="row q-col-gutter-x-md">
              <div class="col-md col-12">
                <q-input v-model="form.name" label="Name *" required />
              </div>
              <div class="col-md col-12">
                <q-input
                  v-model="form.description"
                  type="description"
                  label="description *"
                  required
                />
              </div>
            </div>
          </div>
          <div align="right" class="q-pa-sm q-gutter-sm">
            <q-btn label="Close" @click="close()" />
            <q-btn label="Save" color="teal" type="submit" />
          </div>
        </q-form>
      </q-card-section>
    </q-card>
  </q-dialog>
</template>

<script>
import { api } from 'boot/axios'
import { ref, defineComponent } from 'vue'
import { Notify } from 'quasar'

export default defineComponent({
  name: 'itemdetailsForm',
  data() {
    return {
      name: 'itemdetails Form',
      dialog: ref(false),
      id: null,
      url: '/items/',
      form: {},
    }
  },
  methods: {
    notify(message, color) {
      Notify.create({
        position: 'top',
        progress: true,
        message: message,
        color: color,
      })
    },
    add() {
      this.id = null
      this.dialog = ref(true)
    },
    async edit(id) {
      this.id = id
      try {
        const res = await api.get(this.url + this.id)
        this.form = res.data
        this.dialog = ref(true)
      } catch (err) {
        console.error(err)
      }
    },
    submit() {
      if (this.id) {
        this.update()
      } else {
        this.create()
      }
    },
    async create() {
      try {
        const res = await api.post(this.url, this.form)
        this.emit('doAdd', res.data)
      } catch (err) {
        this.notify('Something went wrong. Try again.', 'red')
      }
    },

    async update() {
      try {
        const res = await api.put(this.url + this.id, this.form)
        this.emit('doEdit', res.data)
      } catch (err) {
        console.log(err)
        // Handle the error and provide appropriate notification
        if (err) {
          this.notify(err, 'red')
        } else {
          this.notify('Try Again', 'red')
        }
      }
    },
    emit(type, res) {
      this.$emit(type, res)
      this.close()
    },
    close() {
      this.id = null
      this.form = {}
      this.dialog = ref(false)
    },
  },
})
</script>
