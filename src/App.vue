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
              <v-col no-gutters v-for="(r, i) in groupByLabel(rIndex, String.fromCharCode(65 + index))" :key="i" class="grid ma-1" :class="{ walkway: r.isWalkWay }">
                <div class="pa-2" :class="{ filled: !r.isAvailable }" :style="{ 'background-color': getDynamicColor(r) }">
                  <div>{{ r.gridLabel }}</div>
                  <div>
                    <span v-if="r.isWalkWay">Walk Way</span>
                    <span v-else>{{ r?.order?.substr(18, 24) }}</span>
                  </div>
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
      storageFacilityId: '63ff2d3c6efab8a1c603a592',
      // storageFacilityId: '63c7c744020d5c9f0f4981ad',
      rooms: null,
      // env: 'https://api.valetcloset.com',
      env: 'https://dev-api.valetcloset.com',
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
    stringToColor(str) {
      var hash = 0;
      for (var i = 0; i < str.length; i++) {
        hash = str.charCodeAt(i) + ((hash << 5) - hash);
      }

      var c = (hash & 0x00ffffff).toString(16).toUpperCase();

      return `#${'00000'.substring(0, 6 - c.length) + c}`;
    },
    getDynamicColor(r) {
      if (r.order) {
        return this.stringToColor(r.order);
      } else {
        return '';
      }
    },
    async getStorageFacilityDetails(storageFacilityId) {
      try {
        const response = await axios.get(`${this.env}/storage-spaces/storage-facilities/${storageFacilityId}`);
        this.rooms = _.groupBy(response?.data, 'room') || [];
      } catch (error) {
        console.error(error);
      }
    },
    groupByLabel(row, char) {
      console.log(row, char, this.rooms);
      return _.filter(
        _.sortBy(this.rooms[row], (r) => Number(r?.gridLabel?.slice(1, 10))),
        (i) => i.gridLabel.startsWith(char)
      );
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.grid {
  border: 1px solid #555;
  border-radius: 4px;
}
.filled {
  color: #fff;
}
.walkway {
  background-image: linear-gradient(to right, red, blue);
}
</style>
