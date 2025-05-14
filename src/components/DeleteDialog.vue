<template>
  <q-dialog v-model="dialog" persistent>
    <q-card style="width: 300px">
      <q-card-section>
        <div class="text-h6 text-center">Are you absolutely sure?</div>
      </q-card-section>

      <q-card-section class="q-pt-none text-center">
        Do you want to delete <strong>{{ name }}</strong
        >?
      </q-card-section>

      <q-card-actions class="q-pb-md" align="center">
        <q-btn color="red" label="yes" @click="close" />
        <q-btn color="green" label="No" v-close-popup />
      </q-card-actions>
    </q-card>
  </q-dialog>
</template>

<script>
import { ref } from 'vue'
export default {
  data() {
    return {
      name: '',
      dialog: ref(false),
    }
  },
  methods: {
    show(name) {
      this.name = name
      this.dialog = ref(true)
    },
    close() {
      this.dialog = ref(false)
      this.$emit('doDelete')
    },
  },
}
</script>
