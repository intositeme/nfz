<template>
  <div class="holder">
    <h1>Maps</h1>
    <gmap-map
      :center="getCenter"
      :zoom="zoom"
      style="width: 100%; min-height: 500px"
    >
      <gmap-marker
        :key="index"
        v-for="(m, index) in nfz"
        :position="m"
        :clickable="true"
        @click="center=m"
      ></gmap-marker>

      <gmap-marker
        v-if="currentLocation"
        :position="currentLocation"
        icon="//chart.apis.google.com/chart?chst=d_map_pin_letter&chld=%E2%80%A2|FFFF00"
        animation=2
      ></gmap-marker>

      <gmap-circle
        :key="index"
        v-for="(m, index) in nfz"
        :center="m"
        :radius="m.radius"
        :options="circleOptions"
      >
      </gmap-circle>
    </gmap-map>
  </div>
</template>
<script>
  import * as VueGoogleMaps from 'vue2-google-maps'
  import Vue from 'vue'
  // http://www.dji.com/api/no-fly/country/SG?v=2
  //
  Vue.use(VueGoogleMaps, {
    load: {
      key: 'AIzaSyAPWVOfR6Sb2_m-qRSr91ph_riIapOD230'
      // v: 'OPTIONAL VERSION NUMBER',
      // libraries: 'places', //// If you need to use place input
    }
  })

  export default {
    data () {
      return {
        center: {lat: 1.352083, lng: 103.819836},
        zoom: 11,
        nfz: [],
        circleOptions: {
          strokeColor: '#FF0000',
          strokeOpacity: 0.8,
          strokeWeight: 2,
          fillColor: '#FF0000',
          fillOpacity: 0.35
        },
        currentLocation: null,
        pinImage: {}
        /*
        markers: [{
          position: {lat: 10.0, lng: 10.0}
        }, {
          position: {lat: 11.0, lng: 11.0}
        }] */
        // http://chart.apis.google.com/chart?chst=d_map_pin_letter&chld=%E2%80%A2|" + pinColor
      }
    },
    computed: {
      getCenter () {
        if (this.currentLocation != null) return this.currentLocation
        return this.center
      }
    },
    mounted () {
      console.log('maps component')
      // this.pinImage = VueGoogleMaps.MarkerImage('http://chart.apis.google.com/chart?chst=d_map_pin_letter&chld=%E2%80%A2|' + 'FFFF00', VueGoogleMaps.Size(21, 34), VueGoogleMaps.Point(0, 0), VueGoogleMaps.Point(10, 34))

      this.$http.get('http://www.dji.com/api/no-fly/country/SG?v=2')
        .then((response) => {
          this.nfz = response.data.data
          this.loaded()
        })
        .catch(function (error) {
          console.log(error)
        })
    },
    methods: {
      loaded () {
        console.log('loaded')
        if (navigator.geolocation) {
          navigator.geolocation.getCurrentPosition(this.locationGot)
        } else {
          console.log('Geolocation is not supported by this browser.')
        }
      },
      locationGot (position) {
        //  position.coords.latitude position.coords.longitude;
        this.currentLocation = {}
        this.currentLocation.lat = position.coords.latitude
        this.currentLocation.lng = position.coords.longitude
      }
    }
  }

</script>
