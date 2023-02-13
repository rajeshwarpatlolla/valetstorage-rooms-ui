<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <div class="d-flex align-center">
        <div>Storage Facility Details</div>
      </div>
    </v-app-bar>

    <v-main>
      <v-row class="pa-8">
        <v-col cols="12" md="4">
          <v-select label="Select Env" :items="envs" item-text="label" v-model="env"></v-select>
          <v-text-field v-model="storageFacilityId" label="Enter storage facility id" required></v-text-field>
          <v-btn type="submit" block class="mt-2" @click="getStorageFacilityDetails(storageFacilityId)">Get Rooms Availability</v-btn>
        </v-col>
      </v-row>

      <div v-if="rooms" class="pa-4">
        <v-row no-gutters class="pt-4" v-for="(room, rIndex) in rooms" :key="rIndex">
          <v-col cols="12">
            <div class="pb-4">Room {{ rIndex }}</div>
            <v-row no-gutters v-for="(row, index) in rows" :key="index">
              <v-col no-gutters v-for="(r, i) in groupByLabel(rIndex, String.fromCharCode(65 + index))" :key="i" class="grid ma-1">
                <div class="pa-2 details" :class="{ filled: !r.isAvailable }">
                  <div>{{ r.label }}</div>
                  <div>{{ r?.order?.substr(12, 24) }}</div>
                </div>
              </v-col>
            </v-row>
          </v-col>
        </v-row>
      </div>
    </v-main>
  </v-app>
</template>

<script>
import _ from 'lodash';
import axios from 'axios';

export default {
  name: 'App',
  data() {
    return {
      rows: 26,
      storageFacilityId: '63e62d9da6945347ad39a511',
      rooms: null,
      env: 'http://localhost:3000',
      envs: [
        { label: 'Localhost', value: 'http://localhost:3000' },
        { label: 'Develop', value: 'https://dev-api.valetcloset.com' },
        { label: 'Staging', value: 'https://stg-api.valetcloset.com' },
        { label: 'Production', value: 'https://api.valetcloset.com' },
      ],
    };
  },
  mounted() {},
  methods: {
    async getStorageFacilityDetails(storageFacilityId) {
      try {
        const response = await axios.get(`${this.env}/storage-spaces/storage-facilities/${storageFacilityId}`);
        this.rooms = _.groupBy(response.data, 'room') || [];
      } catch (error) {
        console.error(error);
      }
    },
    groupByLabel(row, char) {
      // return _.filter(_.sortBy(this.rooms[row], 'label'), (i) => i.label.startsWith(char));
      return _.filter(_.sortBy(this.rooms[row], 'cratedAt'), (i) => i.label.startsWith(char));
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.grid {
  background-color: green;
}
.filled {
  background-color: red;
}
.details {
  color: #fff;
}
</style>
