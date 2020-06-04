<template>
  <div id="app">
    <div class="row no-gutters">
      <!-- 選擇地區 -->
      <div class="toolbox col-sm-3 p-2 bg-white">
        <div class="form-group d-flex">
          <label for="city" class="col-form-label mr-2 text-right">縣市</label>
          <div class="flex-fill">
            <select id="city" class="form-control" v-model="select.city">
              <!-- 製作下拉選單 -->
              <option v-for="city in cities" :key="city.name" :value ="city.name">
                {{ city.name }}
              </option>
            </select>
          </div>
        </div>
        <div class="form-group d-flex">
          <label for="dist" class="col-form-label mr-2 text-right">地區</label>
          <div class="flex-fill">
            <select id="dist" class="form-control" v-model="select.dist">
              <option v-for="district in districts" :key="district.name" :value="district.name">
                {{ district.name }}
              </option>
              <!-- 製作下拉選單 -->
            </select>
          </div>
        </div>
      </div>

      <!-- 顯示地圖和 UBike 站點 -->
      <div class="col-sm-9">
        <div id="map"></div>
      </div>
    </div>
  </div>
</template>

<script>
import L from 'leaflet';
import taipeiCity from './assets/taipei_city.json';

export default {
  name: 'App',
  data: () => ({
    select: {
      city: '臺北市',
      dist: taipeiCity[0].districts[0].name,
    },
    cities: taipeiCity,
    retVal: [],
    OSM: {},
  }),
  computed: {
    districts() {
      return this.cities.find((city) => city.name === this.select.city).districts;
    },
    youbikes() {
      return this.retVal.filter((stop) => stop.sarea === this.select.dist);
    },
  },
  watch: {
    youbikes() {
      this.updateMap();
    },
  },
  methods: {
    updateMap() {
      this.OSM.eachLayer((layer) => {
        if (layer instanceof L.Marker) {
          this.OSM.removeLayer(layer);
        }
      });

      this.youbikes.forEach((bike) => {
        L.marker([bike.lat, bike.lng])
          .bindPopup(`<p><strong style="font-size: 20px;">${bike.sna}</strong></p>
        <strong style="font-size: 16px; color: #d45345;">可租借車輛剩餘：${bike.sbi} 台</strong><br>
        可停空位剩餘: ${bike.bemp}<br>
        <small>最後更新時間: ${bike.mday}</small>`)
          .addTo(this.OSM);
      });

      this.cities[0].districts.forEach((dist) => {
        if (dist.name === this.select.dist) {
          this.OSM.panTo(L.latLng(dist.latitude, dist.longitude));
        }
      });
    },
  },
  created() {
    const api = 'https://tcgbusfs.blob.core.windows.net/blobyoubike/YouBikeTP.json';
    this.$http.get(api).then((response) => {
      this.retVal = Object.values(response.data.retVal);
    });
  },
  mounted() {
    this.OSM = L.map('map', {
      center: [25.044214, 121.524725],
      zoom: 18,
    });

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
      maxZoom: 18,
    }).addTo(this.OSM);
  },
};

</script>

<style lang="scss">
@import 'bootstrap/scss/bootstrap';

#map {
  height: 100vh;
  position: relative;
  background-color: coral;
}
</style>
