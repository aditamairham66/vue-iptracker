<template>
  <div class="flex flex-col h-screen max-h-screen">
    <div class="z-20 flex justify-center relative bg-hero-pattern bg-cover px-4 pt-8 pb-32">

      <div class="w-full max-w-screen-sm">
        <h1 class="text-white text-center text-3xl pb-4">Ip Address Tracker</h1>
        <div class="flex">
          <input type="text" v-model="queryIp" 
            class="flex-1 py-3 px-2 rounded-tl-md rounded-bl-md focus:outline-none" 
            placeholder="Search for any IP address or leave empty to get your ip info">

            <div class="flex items-center cursor-pointer bg-black text-white px-4 rounded-tr-md rounded-br-md" 
              @click="getIpInfo">
              <i class="fas fa-chevron-right"></i>
            </div>
        </div>
      </div>
      
      <IpInfo v-if="ipInfo" v-bind:ipInfo="ipInfo" />

    </div>

    <div id="mapID" class="h-full z-10"></div>

  </div>
</template>

<script>
import IpInfo from '@/components/IpInfo.vue'
import { onMounted, ref } from 'vue';
import leaflet from 'leaflet';
import axios from 'axios';

export default {
  name: 'HomeView',
  components: {
    IpInfo
  },
  setup() {
    let mymap;

    const queryIp = ref("");
    const ipInfo = ref(null);

    onMounted(() => {
      mymap = leaflet.map('mapID').setView([51.505, -0.09], 13);

      leaflet
        .tileLayer(
          "https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=pk.eyJ1Ijoiam9obmtvbWFybmlja2kiLCJhIjoiY2txcGJ3bmoyMHRjMzJ2cGNpMnlsc3pqeiJ9.mSoT7QxEgjVEmNRN9k_NKg",
          {
            attribution:
              'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
            maxZoom: 18,
            id: "mapbox/streets-v11",
            tileSize: 512,
            zoomOffset: -1,
            accessToken:
              "pk.eyJ1Ijoiam9obmtvbWFybmlja2kiLCJhIjoiY2txcGJ3bmoyMHRjMzJ2cGNpMnlsc3pqeiJ9.mSoT7QxEgjVEmNRN9k_NKg",
          }
        )
        .addTo(mymap);
    })

    // gets ip information from API
    const getIpInfo = async () => {
      try {
        const data = await axios.get(
          `https://geo.ipify.org/api/v1?apiKey=at_FBTi0rGPhQlKp2cRdOJV0t3Mibq6F&ipAddress=${queryIp.value}`
        );
        const result = data.data;
        ipInfo.value = {
          address: result.ip,
          state: result.location.region,
          timezone: result.location.timezone,
          isp: result.isp,
          lat: result.location.lat,
          lng: result.location.lng,
        };
        leaflet.marker([ipInfo.value.lat, ipInfo.value.lng]).addTo(mymap);
        mymap.setView([ipInfo.value.lat, ipInfo.value.lng], 13);
      } catch (err) {
        alert(err.message);
      }
    };

    return { queryIp, ipInfo, getIpInfo };
  }
}
</script>
