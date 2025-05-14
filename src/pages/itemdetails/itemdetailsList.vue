<template>
  <q-card>
    <q-table
      :title="name"
      :rows="rows"
      :columns="columns"
      :filter="filter"
      row-key="_id"
      :pagination="initialPagination"
    >
      <template v-slot:top-right>
        <div class="q-gutter-md row items-start">
          <q-input dense debounce="300" v-model="filter" placeholder="Search">
            <template v-slot:append>
              <q-icon name="search" />
            </template>
          </q-input>

          <q-btn color="teal" icon-right="add" label="Add" @click="onAdd" />
        </div>
      </template>
      <template v-slot:body-cell-actions="props">
        <q-td :props="props">
          <q-btn dense round flat color="blue" @click="onEdit(props)" icon="edit">
            <q-tooltip>Edit</q-tooltip>
          </q-btn>
          <q-btn dense round flat color="red" @click="onDelete(props)" icon="delete">
            <q-tooltip>Delete</q-tooltip>
          </q-btn>
        </q-td>
      </template>
    </q-table>
  </q-card>
  <FormDialog ref="formDialog" @doAdd="doAdd" @doEdit="doEdit" />
  <DeleteDialog ref="deleteDialog" @doDelete="doDelete" />
</template>

<script>
import FormDialog from './itemdetailsForm.vue'
import DeleteDialog from 'components/DeleteDialog.vue'
import moment from 'moment'
import _ from 'lodash'
import { api } from 'boot/axios'
import { ref, defineComponent } from 'vue'
import { Notify, exportFile } from 'quasar'

export default defineComponent({
  name: 'itemdetailsList',
  components: {
    FormDialog,
    DeleteDialog,
  },
  data() {
    return {
      name: 'itemdetails',
      url: '/items/',
      filter: ref(''),
      initialPagination: {
        sortBy: 'createdOn',
        descending: true,
        rowsPerPage: 10,
      },
      columns: [
        {
          name: 'name',
          required: true,
          label: 'NAME',
          align: 'left',
          field: 'name',
          sortable: true,
        },
        {
          name: 'description',
          required: true,
          label: 'Description',
          align: 'left',
          field: 'description',
          sortable: true,
        },
        {
          name: 'createdOn',
          align: 'left',
          label: 'Created On',
          field: 'createdOn',
          format: (val) => `${this.getDate(val)}`,
          sortable: true,
        },
        {
          name: 'updatedOn',
          align: 'left',
          label: 'Updated On',
          field: 'updatedOn',
          format: (val) => `${this.getDate(val)}`,
          sortable: true,
        },
        {
          name: 'actions',
          label: 'Actions',
          field: '',
          align: 'left',
        },
      ],
      rows: [],
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
    getDate(date) {
      return moment(date).format('DD/MM/YYYY hh:mm A')
    },
    onLogs(props) {
      console.log(props)
      this.$refs.logsDialog.showSimcard(props.key)
    },
    onAdd() {
      this.$refs.formDialog.add()
    },
    doAdd(res) {
      this.rows.push(res)
      this.notify(res.name + ' added successfully', 'green')
    },
    onEdit(props) {
      this.$refs.formDialog.edit(props.key)
    },
    doEdit(res) {
      this.rows.splice(_.findIndex(this.rows, { _id: res._id }), 1, res)
      this.notify(res.name + ' updated successfully', 'green')
    },
    onDelete(props) {
      this.deleteData = props
      this.$refs.deleteDialog.show(props.row.name)
    },
    async doDelete() {
      try {
        await api.delete(this.url + this.deleteData.key)

        // Find the index of the item to be removed and delete it
        const index = _.findIndex(this.rows, { _id: this.deleteData.key })
        if (index !== -1) {
          this.rows.splice(index, 1)
        }

        this.notify(this.deleteData.row.name + ' deleted successfully', 'red')
        this.deleteData = {}
      } catch (err) {
        console.error(err)
        // Handle or notify about the error if necessary
        if (err.response?.status === 400) {
          this.notify(err.response.data.message, 'red') // Display server-side error message
        }
      }
    },
    wrapCsvValue(val, formatFn, row) {
      let formatted = formatFn !== void 0 ? formatFn(val, row) : val
      formatted = formatted === void 0 || formatted === null ? '' : String(formatted)
      formatted = formatted.split('"').join('""')
      /**
       * Excel accepts \n and \r in strings, but some other CSV parsers do not
       * Uncomment the next two lines to escape new lines
       */
      // .split('\n').join('\\n')
      // .split('\r').join('\\r')
      return `"${formatted}"`
    },
    exportTable() {
      // Filter out the 'actions' column
      const filteredColumns = this.columns.filter((col) => col.name !== 'actions')

      // Naive encoding to csv format
      const content = [filteredColumns.map((col) => this.wrapCsvValue(col.label))]
        .concat(
          this.rows.map((row) =>
            filteredColumns
              .map((col) =>
                this.wrapCsvValue(
                  typeof col.field === 'function'
                    ? col.field(row)
                    : row[col.field === void 0 ? col.name : col.field],
                  col.format,
                  row,
                ),
              )
              .join(','),
          ),
        )
        .join('\r\n')

      const status = exportFile(this.name + '-table-export.csv', content, 'text/csv')

      if (status !== true) {
        this.notify({
          message: 'Browser denied file download...',
          color: 'negative',
          icon: 'warning',
        })
      }
    },
  },
  mounted() {
    api
      .get(this.url)
      .then((res) => {
        this.rows = res.data
      })
      .catch((err) => {
        console.log(err)
      })
  },
})
</script>
