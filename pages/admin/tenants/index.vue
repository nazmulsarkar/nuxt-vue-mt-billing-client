<template>
  <div class="container-fluid">
    <div class="text-center">
      <h3 class="title">
        Tenant datatable using vuetify
      </h3>
      <hr>
      <div v-if="datas.length === 0">
        <h2>No tenant found at the moment</h2>
      </div>
    </div>
    <!-- @update:pagination="onChangePagination" -->
    <div id="app">
      <v-app>
        <v-data-table
          v-model="selected"
          :headers="headers"
          :items="datas"
          :options.sync="options"
          :server-items-length="totalTenants"
          :loading="loading"
          :dense="dense"
          :page="page"
          :items-per-page="itemsPerPage"
          :single-select="singleSelect"
          sort-by="create_at"
          item-key="_id"
          show-select
          class="elevation-1"
        >
          <template v-slot:item.thumb_url="{ item }">
            <div class="p-2">
              <v-img :src="item.thumb_url" :alt="item.title" width="100px" />
            </div>
          </template>
          <template v-slot:top>
            <v-toolbar flat color="white">
              <v-toolbar-title>Tenant CRUD</v-toolbar-title>
              <v-divider class="mx-4" inset vertical />
              <v-spacer />
              <v-dialog v-model="dialog" max-width="500px">
                <template v-slot:activator="{ on }">
                  <v-btn v-on="on" color="primary" dark class="mb-2">
                    Create Tenant
                  </v-btn>
                </template>
                <v-card>
                  <v-card-title>
                    <span class="headline">{{ formTitle }}</span>
                  </v-card-title>

                  <v-card-text>
                    <v-container>
                      <v-row>
                        <v-col cols="12">
                          <v-text-field v-model="editedItem.name" label="Tenant name" />
                        </v-col>
                        <v-col cols="12">
                          <v-text-field v-model="editedItem.contact_email" label="Contact email" />
                        </v-col>
                      </v-row>
                    </v-container>
                  </v-card-text>

                  <v-card-actions>
                    <v-spacer />
                    <v-btn @click="close" color="blue darken-1" text>
                      Cancel
                    </v-btn>
                    <v-btn @click="save" color="blue darken-1" text>
                      Save
                    </v-btn>
                  </v-card-actions>
                </v-card>
              </v-dialog>
            </v-toolbar>
          </template>
          <template v-slot:item.action="{ item }">
            <v-icon @click="editItem(item)" small class="mr-2">
              edit
            </v-icon>
            <v-icon @click="deleteItem(item)" small>
              delete
            </v-icon>
          </template>
          <template v-slot:no-data>
            <v-btn @click="fetchData" color="primary">
              Reset
            </v-btn>
          </template>
          <!-- <v-pagination :length="pagination.total / 5" v-model="pagination.page" :total-visible="pagination.visible"></v-pagination> -->
        </v-data-table>
      </v-app>
    </div>
  </div>
</template>
<script>
import axios from 'axios'
// import { server } from '../helper'
export default {
  layout: 'admin-layout',
  data: () => ({
    totalTenants: 0,
    page: 1,
    itemsPerPage: 5,
    loading: true,
    dense: true,
    options: {},
    dialog: false,
    singleSelect: true,
    selected: [],
    headers: [
      { text: 'Tenant Name', align: 'left', sortable: true, value: 'name' },
      { text: 'Contact Email', sortable: true, value: 'contact_email' },
      { text: 'Created', value: 'created_at' },
      { text: 'Actions', value: 'action', sortable: false }
    ],
    datas: [],
    editedIndex: -1,
    editedItem: {
      name: '',
      contact_email: ''
    },
    defaultItem: {
      name: '',
      contact_email: ''
    }
  }),
  computed: {
    formTitle () {
      return this.editedIndex === -1 ? 'Create Tenant' : 'Edit Tenant'
    },
    url () {
      return `${process.env.API_BASE_URL}/tenants?page=${this.page}&limit=${this.itemsPerPage}`
    }
  },
  watch: {
    dialog (val) {
      val || this.close()
    },
    options: {
      handler () {
        this.loading = true
        setTimeout(() => {
          this.fetchData()
        }, 500)
      },
      deep: true
    }
  },
  // created () {
  //   this.fetchProducts()
  // },
  methods: {
    fetchData () {
      const { page, itemsPerPage } = this.options
      this.page = page
      this.itemsPerPage = itemsPerPage
      axios.get(this.url).then((response) => {
        // console.log(response)
        this.datas = response.data.data
        this.totalProducts = response.data.total
        this.loading = false
      })
    },
    // onChangePagination (pagination) {
    //   console.log('update:pagination', pagination)
    // },
    editItem (item) {
      this.editedIndex = this.products.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },

    deleteItem (item) {
      const index = this.datas.indexOf(item)
      if (confirm('Are you sure you want to delete this tenant?')) {
        axios
          .delete(`${process.env.API_BASE_URL}/tenants/${item._id}`)
          .then((data) => {
            this.datas.splice(index, 1)
            this.$toast.show(
              "Success! '" + item.title + "' tenant deleted...",
              {
                position: 'bottom-center',
                duration: 3000
              }
            )
          })
      }
    },

    close () {
      this.dialog = false
      setTimeout(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      }, 300)
    },

    save () {
      if (this.editedIndex > -1) {
        Object.assign(this.datas[this.editedIndex], this.editedItem)
        const payload = this.editedItem
        axios
          .put(`${process.env.API_BASE_URL}/tenants/${payload._id}`, payload)
          .then((data) => {
            // console.log(data);
            this.$toast.show(
              "Success! Tenant '" + payload.title + "' updated...",
              {
                position: 'bottom-center',
                duration: 3000
              }
            )
          })
      } else {
        const payload = this.editedItem
        axios
          .post(`${process.env.API_BASE_URL}/tenants`, payload)
          .then((data) => {
            // console.log(data);
            this.products.push(this.editedItem)
            this.$toast.show(
              "Success! New tenant '" + payload.title + "' created.",
              {
                position: 'bottom-center',
                duration: 3000
              }
            )
          })
      }
      this.close()
    }
  }
}
</script>

<style scoped>
.v-dialog .container {
  min-height: inherit;
}
</style>
