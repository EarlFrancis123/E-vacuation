<template>
  <v-container>
    <v-card>
      <v-card-title>
        Evacuation Centers
        <v-spacer></v-spacer>

        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on }">
            <v-btn color="wizniche-blue" dark class="mb-2" v-on="on">Add New Evacuation Center</v-btn>
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
                      v-model="editedItem.evacuationName"
                      label="Evacuation Center Name"
                      :rules="[required('Evacuation Center Name')]"
                    ></v-text-field>
                  </v-row>
                  <v-row>
                    <v-text-field
                      v-model="editedItem.evacuationBarangay"
                      label="Barangay"
                      :rules="[required('Barangay')]"
                    ></v-text-field>
                  </v-row>
                  <v-row>
                    <v-autocomplete
                      v-model="editedItem.evacuationCalamityType"
                      :items="calamities"
                      :search-input.sync="search"
                      item-text="calamityName"
                      item-value="calamityName"
                      label="Calamity Type"
                      :rules="[required('Calamity Type')]"
                    ></v-autocomplete>
                  </v-row>
                  <v-row>
                    <v-text-field
                      v-model="editedItem.evacuationNumber"
                      label="Contact Number"
                      :rules="[required('Contact Number')]"
                    ></v-text-field>
                  </v-row>
                  <v-row>
                    <v-text-field
                      v-model="editedItem.state"
                      label="City"
                      :rules="[required('City')]"
                    ></v-text-field>
                  </v-row>
                  <v-row>
                      <GmapAutocomplete @place_changed="setPlace" />
                  </v-row>
                  <v-row>
                  <input type="file" accept="image/jpeg" @change="uploadImage">
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
        :items="evacuations"
        :search="search"
        sort-by="name"
        class="elevation-1"
      >
        <template v-slot:top>
          <v-toolbar flat>
            <v-text-field
              v-model="search"
              append-icon="mdi-magnify"
              label="Search Evacuation Center"
              single-line
              hide-details
            ></v-text-field>
          </v-toolbar>
        </template>
        <template v-slot:[`item.image`]="{ item }">
         <img :src="item.image" width="25" height="25">
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
    previewImage: null,
    headers: [
      {
        text: 'Evacuation Center',
        align: 'start',
        sortable: false,
        value: 'evacuationName',
      },
      {
        text: 'Barangay',
        align: 'start',
        sortable: false,
        value: 'evacuationBarangay',
      },
      {
        text: 'Calamity Type',
        align: 'start',
        sortable: false,
        value: 'evacuationCalamityType',
      },
      {
        text: 'Contact Number',
        align: 'start',
        sortable: false,
        value: 'evacuationNumber',
      },
      {
        text: 'City',
        align: 'start',
        sortable: false,
        value: 'state',
      },
      {
        text: 'Street Address',
        align: 'start',
        sortable: false,
        value: 'streetAddress',
      },
      { text: 'Actions', value: 'actions', sortable: false, width: '150' },
    ],
    editedIndex: -1,
    editedItem: {
      evacuationName: '',
      country: '',
      evacuationBarangay: '',
      latitude: '',
      longitude: '',
      streetAddress: '',
      image: ''
    },
    defaultItem: {
      evacuationName: '',
      country: '',
      evacuationBarangay: '',
      latitude: '',
      longitude: '',
      streetAddress: '',
      image: ''
    },
    center: { lat: 45.508, lng: -73.587 },
    currentPlace: null,
    markers: [],
    ...validations
  }),
  async fetch({ store }) {
    try {
      await store.dispatch('evacuation/loadAllEvacuations');
      await store.dispatch('calamity/loadAllCalamities');
    } catch (e) { }
  },
  computed: {
    ...mapState({
      evacuations: state => state.evacuation.evacuations,
      calamities: state => state.calamity.calamities
    }),
    formTitle () {
      return this.editedIndex === -1 ? 'New Evacuation Center' : 'Edit Evacuation Center'
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
      this.editedIndex = this.evacuations.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },
    async deleteItem (item) {
      try {
        confirm('Are you sure you want to delete this item?') && await this.$store.dispatch('evacuation/delete', item);
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
          await this.$store.dispatch('evacuation/edit', this.editedItem);
        } else {
          await this.$store.dispatch('evacuation/create', this.editedItem);
        }
      } catch (e) { }
      this.close()
    },
    setPlace(place) {
      this.currentPlace = place;
      this.editedItem.latitude = this.currentPlace.geometry.location.lat();
      this.editedItem.longitude = this.currentPlace.geometry.location.lng();
      this.editedItem.streetAddress = this.currentPlace.formatted_address;
    },
    addMarker() {
      if (this.currentPlace) {
        const marker = {
          lat: this.currentPlace.geometry.location.lat(),
          lng: this.currentPlace.geometry.location.lng(),
        };
        this.markers.push({ position: marker });
        this.center = marker;
        this.currentPlace = null;
      }
    },
    uploadImage(e){
        const image = e.target.files[0];
        const reader = new FileReader();
        reader.readAsDataURL(image);
        reader.onload = e =>{
        this.previewImage = e.target.result;
        this.editedItem.image = this.previewImage.replace(/^data:image\/[a-z]+;base64,/, "");
      };
    },
  },
}
</script>

