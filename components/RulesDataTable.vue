<template>
  <v-container>
    <v-data-table
      :headers="headers"
      :items="rules"
      hide-default-footer
      sort-by="calories"
      class="elevation-1"
    >
      <template v-slot:top>
        <v-toolbar flat >
          <v-toolbar-title>SAS CRUD</v-toolbar-title>
          <v-divider
            class="mx-4"
            inset
            vertical
          ></v-divider>
          <v-spacer></v-spacer>
          <v-dialog v-model="dialog" max-width="500px">

            <template v-slot:activator="{ on, attrs }">
              <v-btn
                color="primary"
                dark
                class="mb-2"
                v-bind="attrs"
                v-on="on"
              >New Item</v-btn>
            </template>

            <v-card>
              <v-card-title>
                <span class="headline">{{ formTitle }}</span>
              </v-card-title>

              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col v-if="editedIndex !== -1"  cols="12" sm="6" md="6">
                      <v-text-field disabled v-model="item.house_rules.id" label="Id"></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="6">
                      <v-text-field :disabled="editedIndex === -2" v-model="item.house_rules.name" label="Name"></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="6">
                      <v-text-field :disabled="editedIndex === -2" v-model="item.house_rules.active" label="Active"></v-text-field>
                    </v-col>
                    <v-col v-if="editedIndex !== -1" cols="12" sm="6" md="6">
                      <v-text-field disabled v-model="item.house_rules.order" label="Order"></v-text-field>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="close">Cancel</v-btn>
                <v-btn v-if="editedIndex !== -2" color="blue darken-1" text @click="save">Save</v-btn>
              </v-card-actions>
            </v-card>

          </v-dialog>
        </v-toolbar>
      </template>

      <template v-slot:item.actions="{ item }">
        <v-icon
          small
          class="mr-2"
          @click="editItem(item)"
        >
          mdi-pencil
        </v-icon>
        <v-icon
          small
          class="mr-2"
          @click="deleteItem(item)"
        >
          mdi-delete
        </v-icon>
        <v-icon
          small
          @click="show(item)"
        >
          mdi-eye
        </v-icon>
      </template>

      <template v-slot:no-data>
        <v-btn color="primary" @click="getRules">Reset</v-btn>
      </template>


      <template v-slot:bottom>
      </template>
    </v-data-table>

    <div class="text-right pt-2">
      <v-btn
        v-if="pagination.prev"
        rounded
        @click="getRules(pagination.prev.replace('http://', 'https://'))"
      > 
        <v-icon>mdi-chevron-left</v-icon>
      </v-btn>
      <v-btn
        v-if="pagination.next"
        rounded
        @click="getRules(pagination.next.replace('http://', 'https://'))"
      > 
        <v-icon>mdi-chevron-right</v-icon>
      </v-btn>
    </div>
  </v-container>
</template>

<script>
  export default {
    data: () => ({
      rules: [],
      pagination: {
        next: null,
        prev: null
      },
      dialog: false,
      headers: [
        { text: 'Active', value: 'active' },
        { text: 'ID', value: 'id' },
        { text: 'Name', value: 'name' },
        { text: 'Order', value: 'order' },
        { text: 'Actions', value: 'actions', sortable: false },
      ],
      editedIndex: -1,
      item: {
        house_rules: {
          id: 0,
          name: '',
          active: null,
          order: 0
        }
      },
      defaultItem: {
        name: '',
        active: null
      },
    }),

    computed: {
      formTitle () {
        if (this.editedIndex === -1) return 'New Item'
        if (this.editedIndex === -2) return 'Show Item'
        return 'Edit Item'
      },
    },

    watch: {
      dialog (val) {
        val || this.close()
      },
    },
    mounted(){
      this.getRules()
    },
    methods: {
      async show (item) {
        this.editedIndex = -2
        await this.$axios.get(`/house_rules/${item.id}`, { headers: { 'Authorization': `Bearer ${localStorage.token}` } })
        this.item.house_rules = Object.assign({}, item)
        this.dialog = true
      },
      async getRules(url) {
        try {
          let rs = await this.$axios.get(url ? url : '/house_rules', { headers: { 'Authorization': `Bearer ${localStorage.token}` } })
          this.rules = rs.data.data.entities
          this.pagination.next = rs.data.data.pagination.links.next
          this.pagination.prev = rs.data.data.pagination.links.prev
        } catch (error) {
          this.$notifier.showMessage({ content: 'Failed to get rules', color: 'error' })
        }
      },

      editItem (item) {
        console.log(item)
        this.editedIndex = this.rules.indexOf(item)
        this.item.house_rules = Object.assign({}, item)
        this.dialog = true
      },

      async deleteItem (item) {
        try {
          await this.$axios.delete(`/house_rules/${item.id}`, { headers: { 'Authorization': `Bearer ${localStorage.token}` } })
          this.$notifier.showMessage({ content: 'Rule deleted successfully', color: 'success' })
        } catch (error) {
          this.$notifier.showMessage({ content: 'Could not delete rule, try again later', color: 'error' })
        } finally {
          this.getRules()
        }
      },

      close () {
        this.dialog = false
        this.$nextTick(() => {
          this.item.house_rules = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        })
      },

      async save () {
        if (this.editedIndex > -1) {
          // update
          try {
            await this.$axios.put(`/house_rules/${this.item.house_rules.id}`, this.item, { headers: { 'Authorization': `Bearer ${localStorage.token}` } })
            this.$notifier.showMessage({ content: 'Rule updated successfully', color: 'success' })
          } catch (error) {
              this.$notifier.showMessage({ content: 'Failed to update rule, try again later', color: 'error' })
          } finally {
            this.getRules()
          }
        } else {
          // create
          this.rules.push(this.item.house_rules)
          try {
            await this.$axios.post('/house_rules', this.item, { headers: { 'Authorization': `Bearer ${localStorage.token}` } })
              this.$notifier.showMessage({ content: 'Rule created successfully', color: 'success' })
          } catch (error) {
              this.$notifier.showMessage({ content: 'Failed to create rule, try again later', color: 'error' })
          } finally {
            this.getRules()
          }
        }
        this.close()
      },
    },
  }
</script>