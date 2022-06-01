<template>
  <v-container>
    <v-card>
      <v-card-title>
        Calamities
        <v-spacer></v-spacer>

        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on }">
            <v-btn color="wizniche-blue" dark class="mb-2" v-on="on">Add New Calamity</v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="headline">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-form v-model="valid">
                  <v-row>
                    <v-text-field
                      v-model="editedItem.calamityName"
                      label="Calamity Name"
                      :rules="[required('Calamity Name')]"
                    ></v-text-field>
                  </v-row>
                  <v-row>
                    <v-text-field
                      v-model="editedItem.calamityDetails"
                      label="Calamity Details"
                      :rules="[required('Calamity Details')]"
                    ></v-text-field>
                  </v-row>
                   <v-row>
                    <v-text-field
                      v-model="editedItem.edCalamityDate"
                      label="Calamity Date (mm/dd/yyyy)"
                      :rules="[required('Calamity Date')]"
                    ></v-text-field>
                  </v-row>
                </v-form>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="close">Cancel</v-btn>
              <v-btn color="blue darken-1" text @click="save" :disabled="!valid">Save</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>

      </v-card-title>

      <v-data-table
        :headers="headers"
        :items="calamities"
        :search="search"
        sort-by="name"
        class="elevation-1"
      >
        <template v-slot:top>
          <v-toolbar flat>
            <v-text-field
              v-model="search"
              append-icon="mdi-magnify"
              label="Search Calamity"
              single-line
              hide-details
            ></v-text-field>
          </v-toolbar>
        </template>
        <template v-slot:[`item.actions`]="{ item }">
          <v-icon
            small
            class="mr-2"
            @click="editItem(item)"
          >
            mdi-pencil
          </v-icon>
          <v-icon
            small
            @click="deleteItem(item)"
          >
            mdi-delete
          </v-icon>
        </template>
      </v-data-table>
    </v-card>
  </v-container>
</template>

<script>
import validations from "~/utils/validations";
import { mapState } from 'vuex';

export default {
  layout ({ layout }) {
    return layout;
  },
  data: () => ({
    valid: false,
    search: '',
    dialog: false,
    headers: [
      {
        text: 'Calamity',
        align: 'start',
        sortable: false,
        value: 'calamityName',
      },
      {
        text: 'Calamity Details',
        align: 'start',
        sortable: false,
        value: 'calamityDetails',
      },
      {
        text: 'Calamity Date',
        align: 'start',
        sortable: false,
        value: 'edCalamityDate',
      },
      { text: 'Actions', value: 'actions', sortable: false, width: '150' },
    ],
    editedIndex: -1,
    editedItem: {
      calamityName: ''
    },
    defaultItem: {
      calamityName: ''
    },
    ...validations
  }),
  async fetch({ store }) {
    try {
      await store.dispatch('calamity/loadAllCalamities');
    } catch (e) { }
  },
  computed: {
    ...mapState({
      calamities: state => state.calamity.calamities
    }),
    formTitle () {
      return this.editedIndex === -1 ? 'New Calamity' : 'Edit Calamity'
    },
  },
  watch: {
    dialog (val) {
      val || this.close()
    },
  },
  created () {
  },
  methods: {
    editItem (item) {
      this.editedIndex = this.calamities.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },
    async deleteItem (item) {
      try {
        confirm('Are you sure you want to delete this item?') && await this.$store.dispatch('calamity/delete', item);
      } catch (e) { }
    },
    close () {
      this.dialog = false
      setTimeout(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      }, 300)
    },
    async save () {
      try {
        if (this.editedIndex > -1) {
          await this.$store.dispatch('calamity/edit', this.editedItem);
        } else {
          await this.$store.dispatch('calamity/create', this.editedItem);
        }
      } catch (e) { }
      this.close()
    },
  },
}
</script>
