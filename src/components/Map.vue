<template>
  <div class="holder">
    <div class="overlay">
      <div class="container-fluid">
        <div class="row justify-content-end" >
          <div class="col-8">
            <div class="list-holder"><v-select :options="countryList" :value.sync="selectedCountry" :on-change="onSelectChange"></v-select></div>
            
          </div>
        </div>
      </div>
      
    </div>
    
    <gmap-map
      ref="map"
      :center="getCenter"
      :zoom="zoom"
      style="width: 100%; min-height: 100vh"
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
  import nfzData from '@/components/nfz-sg'
  import countries from '@/components/countries'
  // http://www.dji.com/api/no-fly/country/SG?v=2
  //
  Vue.use(VueGoogleMaps, {
    load: {
      key: 'AIzaSyAPWVOfR6Sb2_m-qRSr91ph_riIapOD230'
      // v: 'OPTIONAL VERSION NUMBER',
      // libraries: 'places', //// If you need to use place input
    }
  })

  const countryList = []

  for (let key in countries) {
    countryList.push({label: countries[key], value: key})
  }

  export default {
    data () {
      return {
        center: {lat: 1.352083, lng: 103.819836},
        zoom: 11,
        nfz: [],
        countries: countries,
        countryList: countryList,
        circleOptions: {
          strokeColor: '#FF0000',
          strokeOpacity: 0.8,
          strokeWeight: 2,
          fillColor: '#FF0000',
          fillOpacity: 0.35
        },
        currentLocation: null,
        pinImage: {},
        selectedCountry: null,
        geocoder: {}
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
      VueGoogleMaps.loaded.then(() => {
        // console.log('VueGoogleMaps.loaded', google)
        this.geocoder = new google.maps.Geocoder()
      })
      console.log('maps component', VueGoogleMaps.loaded)
      this.nfz = nfzData.data
      this.loaded()
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
      },
      onSelectChange (val) {
        console.log('onSelect change', val)
        let countrycode = val.value
        this.$http.get(`assets/json/${countrycode}.json`)
          .then((response) => {
            this.nfz = response.data.data
            // this.loaded()
            this.geocoder.geocode({ 'address': val.label }, (results, status) => {
              if (status === google.maps.GeocoderStatus.OK) {
                // map.setCenter(results[0].geometry.location)
                console.log('changed country recenter found', results[0], this.$refs)
                this.$refs.map.$mapObject.setCenter(results[0].geometry.location)
                this.$refs.map.$mapObject.fitBounds(results[0].geometry.viewport)
                // this.center.lat = results[0].geometry.location.lat
                // this.center.lng = results[0].geometry.location.lng
              }
            })
          })
          .catch(function (error) {
            console.log(error)
          })
      }
    }
  }

</script>

<style lang="scss">
  .overlay {
    width: 100%;
    position: fixed;
    z-index: 10;
    top: 0px;
    left: 0px;
  }

  .v-select.dropdown {
    background-color: white;
  }
  .list-holder {
    padding-top: 9px;
  }
</style>
