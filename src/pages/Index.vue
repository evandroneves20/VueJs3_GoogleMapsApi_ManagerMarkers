<template>
  <q-page class="q-pa-sm">
    <div class="text-h6">Google Maps</div>
    <div class="text-caption">Click on the map to create a marker or press the New Marker button.</div>
    <q-separator/>

    <div ref="mapDiv" style="width: 100%; height: 60vh"></div>

    <div class="q-my-sm">
      <q-btn outline color="primary" icon="place" @click="createMarker(null)">New Marker</q-btn>
    </div>
    <div>
      <div class="text-h6">Markers</div>
      <template v-if="markers.length > 0">
        <q-list class="q-mb-sm" bordered padding v-for="marker in markers" :key="marker.timestamp">
          <q-item>
            <q-item-section top avatar>
              <q-avatar text-color="red" font-size="2rem" icon="place"/>
            </q-item-section>

            <q-item-section>
              <q-item-label>{{ marker.timestamp }}</q-item-label>
              <q-item-label caption lines="2">Lat: {{ marker.lat.toFixed(4) }} | Lng: {{ marker.lng.toFixed(4) }}
                |
                Alt:
                {{ marker.alt ? marker.alt : '' }}
              </q-item-label>
            </q-item-section>

            <q-item-section top side>
              <div class="text-grey-8 q-gutter-xs">
                <q-btn size="12px" @click="dropMarker(marker)" text-color="red" flat dense round icon="delete"/>
              </div>
            </q-item-section>
          </q-item>
        </q-list>
      </template>
      <template v-else>
        <q-banner inline-actions rounded class="bg-orange text-white">
          No markers
        </q-banner>
      </template>
    </div>
  </q-page>
</template>

<script>
import {defineComponent, ref, onMounted, reactive} from 'vue'
import {useQuasar} from 'quasar'
import {useRoute} from 'vue-router'
import {Loader} from '@googlemaps/js-api-loader'

export default defineComponent({
  name: 'Index',
  setup() {
    const $q = useQuasar()
    const GOOGLE_MAPS_API_KEY = '[YOUR APP KEY]'
    const markers = ref([])
    const tab = ref('mapa')
    let gMarkers = []
    let location = {
      lat: -7,
      lng: -30,
      alt: 0,
      timestamp: 0
    }

    const mapDiv = ref(null)
    let map = null
    let infoWindow = null

    async function loadMap() {
      const loader = new Loader({
        apiKey: GOOGLE_MAPS_API_KEY,
        version: 'weekly',
        libraries: ['places']
      })

      await loader.load()

      // eslint-disable-next-line no-undef
      map = new google.maps.Map(mapDiv.value, {
        center: location,
        zoom: 15,
        disableDefaultUI: true
      })

      // eslint-disable-next-line no-undef
      infoWindow = new google.maps.InfoWindow()

      getLocation()

      // eslint-disable-next-line no-undef
      google.maps.event.addListener(map, 'click', (event) => {
        const gMarker = {
          lat: event.latLng.lat(),
          lng: event.latLng.lng(),
          alt: 0
        }
        createMarker(gMarker)
      })

      loadMarkersOnMap()
      setLines()
    }

    function getLocation() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition((position) => {
          const pos = {
            lat: position.coords.latitude,
            lng: position.coords.longitude,
            alt: position.coords.altitude ?? 0,
            timestamp: position.timestamp
          }
          location = pos
          // infoWindow.setPosition(pos)
          infoWindow.open(map)
          map.setCenter(pos)
        }, (e) => {
          console.log(e)
          handleLocationError(false, infoWindow, map.getCenter())
        })
      }
    }

    function handleLocationError(browserHasGeolocation, infoWindow, pos) {
      infoWindow.setPosition(pos)
      infoWindow.setContent(
        browserHasGeolocation
          ? 'Error: The Geolocation service failed.'
          : 'Error: Your browser doesn\'t support geolocation.'
      )
      infoWindow.open(map)
    }

    function loadStorageMarkers() {
      markers.value = $q.localStorage.getItem('markers') ?? []
    }

    function setLines() {
      // eslint-disable-next-line no-unused-vars,no-undef,no-new
      const flightPath = new google.maps.Polyline({
        path: markers.value,
        geodesic: true,
        strokeColor: '#FF0000',
        strokeOpacity: 1.0,
        strokeWeight: 2
      })

      flightPath.setMap(map)
    }

    function storeLocalMarkers() {
      $q.localStorage.set('markers', markers.value)
    }

    onMounted(() => {
      loadStorageMarkers()
      loadMap()
    })

    function clearMarker(pos) {
      for (let i = 0; i < gMarkers.length; i++) {
        gMarkers[i].setMap(null)
      }

      gMarkers = []
      loadMarkersOnMap()
    }

    function createMarker(pos) {
      if (!pos) { // se for marca no botão
        pos = location
      } else {
        pos.timestamp = Date.now()
      }
      // eslint-disable-next-line no-undef,no-new
      const marker = new google.maps.Marker({
        position: pos,
        map,
        title: 'marker'
      })
      markers.value.push(pos)
      gMarkers.push(marker)

      storeLocalMarkers()
      setLines()
    }

    function loadMarkersOnMap() {
      if (markers.value) {
        markers.value.forEach((value) => {
          // eslint-disable-next-line no-unused-vars,no-undef,no-new
          const marker = new google.maps.Marker({
            position: value,
            map,
            optimized: false
          })
          gMarkers.push(marker)
        })
      }
    }

    function dropMarker(marker) {
      const index = markers.value.findIndex(m => m.lat === marker.lat && m.lng === marker.lng)
      markers.value.splice(index, 1)
      clearMarker()
      storeLocalMarkers()
      loadMap()
    }

    return {
      tab,
      mapDiv,
      loadMap,
      location,
      GOOGLE_MAPS_API_KEY,
      createMarker,
      markers,
      dropMarker,
      gMarkers,
    }
  }
})
</script>

<style scoped>

</style>
