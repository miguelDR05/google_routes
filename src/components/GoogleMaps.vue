<template>
  <!--
      :disable-default-ui="true"
    :map-type-control="false"
    :scale-control="false"
    :street-view-control="false"
    :rotate-control="false"
    :fullscreen-control="false"
  -->
  <GoogleMap
    id="map"
    class="map"
    :api-key="googleMapsApiKey"
    style="width: 100%; height: 500px"
    :center="center"
    :zoom="zoom"
    ref="mapRef"
    @click="addMarker"
    @ready="onMapLoad"
  >
    <!-- <Marker :options="{ position: center }">
      <InfoWindow>
        <div id="content">
          <h6 id="firstHeading" class="firstHeading">Denver</h6>
          <div id="bodyContent">
            <p>
              Denver is a consolidated city and county, the capital, and most
              populous city of the U.S. state of Colorado
            </p>
            <q-btn
              label="Clicl Me"
              @click="
                () => {
                  console.log('click desde Info Window');
                }
              "
            ></q-btn>
          </div>
        </div>
      </InfoWindow>
    </Marker> -->
    <!-- <MarkerCluster>
      <Marker
        v-for="(location, i) in locations"
        :options="{ position: location }"
        :key="i"
      >
      </Marker>
    </MarkerCluster> -->
    <Polyline :options="flightPath" @click="onClickPolimarket" />
    <Marker
      v-for="(marker, index) in markers"
      :key="index"
      :options="{ position: marker }"
      @click="selectMarker(index)"
    />
  </GoogleMap>
  <!-- <label for="lng">Longitude</label>
  <input
    v-model.number="lng"
    id="lng"
    type="number"
    min="-180"
    max="180"
    step="10"
  /> -->
  <div class="row justify-around full-width">
    <q-btn @click="calculateRoute" :disabled="markers.length < 2">
      Calcular Ruta
    </q-btn>
    <q-btn
      @click="removeSelectedMarker"
      :disabled="selectedMarkerIndex === null"
    >
      Eliminar Marcador
    </q-btn>
  </div>
  {{ flightPath.path }}
</template>

<script setup lang="ts">
import { ref, onMounted /*watch, computed*/ } from 'vue';
import { GoogleMap, Polyline, Marker } from 'vue3-google-map'; //

interface Coordinates {
  lat: number;
  lng: number;
}

const googleMapsApiKey = 'AIzaSyBhkuXoZ4HxXbSOcGUzZAvdRgmzWJbzxbA';
const center = ref<Coordinates>({ lat: 37.772, lng: -122.214 });
const zoom = ref(10);
const markers = ref<Coordinates[]>([]);
const mapRef = ref<google.maps.Map | null>(null);

// const flightPlanCoordinates = [
//   { lat: -6.7189973, lng: -79.7681191 },
//   { lat: -6.714111889442064, lng: -79.77356934871827 },
//   { lat: -6.717415007731697, lng: -79.77120900478516 },
//   { lat: -6.71852314561797, lng: -79.7693421873108 },
// ];

const flightPath = ref({
  path: [] as Coordinates[],
  geodesic: true,
  map: mapRef.value,
  strokeColor: '#FFEE00',
  strokeOpacity: 1.0,
  strokeWeight: 2,
  visible: true,
});

const addMarker = (event: google.maps.MapMouseEvent) => {
  if (event.latLng) {
    markers.value.push({
      lat: event.latLng.lat(),
      lng: event.latLng.lng(),
    });
    updateFlightPath();
  }
};

const updateFlightPath = () => {
  flightPath.value.path = markers.value;
};

const selectedMarkerIndex = ref<number | null>(null);

const directionsRenderer = ref<google.maps.DirectionsRenderer | null>(null);

const onMapLoad = (mapInstance: google.maps.Map) => {
  mapRef.value = mapInstance;
  directionsRenderer.value = new google.maps.DirectionsRenderer();
  directionsRenderer.value.setMap(mapInstance);
};

const selectMarker = (index: number) => {
  selectedMarkerIndex.value = index;
};

const removeSelectedMarker = () => {
  if (selectedMarkerIndex.value !== null) {
    markers.value.splice(selectedMarkerIndex.value, 1);
    selectedMarkerIndex.value = null;
    updateFlightPath();
  }
};

const getCurrentLocation = () => {
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition((position) => {
      center.value = {
        lat: position.coords.latitude,
        lng: position.coords.longitude,
      };
      // markers.value.push();
      markers.value.push({
        lat: position.coords.latitude,
        lng: position.coords.longitude,
      });
      updateFlightPath();
    });
  } else {
    console.log('La geolocalizacion no esta soportada en este buscador');
  }
};

const calculateRoute = () => {
  if (markers.value.length < 2) {
    console.error('Necesitas al menos 2 puntos para calcular la ruta.');
    return;
  }

  if (!mapRef.value) {
    console.error('El mapa no está disponible.');
    return;
  }

  const directionsService = new google.maps.DirectionsService();
  directionsRenderer.value?.setMap(mapRef.value);

  const origin = markers.value[0];
  const destination = markers.value[markers.value.length - 1];
  const waypoints = markers.value
    .slice(1, markers.value.length - 1)
    .map((marker) => ({ location: marker, stopover: true }));

  directionsService.route(
    {
      origin,
      destination,
      waypoints,
      optimizeWaypoints: true,
      travelMode: google.maps.TravelMode.DRIVING,
    },
    (response, status) => {
      if (status === google.maps.DirectionsStatus.OK && response) {
        directionsRenderer.value?.setDirections(response);

        const route = response.routes[0];
        const distance = route.legs.reduce(
          (acc, leg) => acc + (leg.distance?.value ?? 0),
          0
        );
        console.log(`Distancia total: ${distance / 1000} km`);

        if (route) {
          flightPath.value.path = route.overview_path.map((point) => ({
            lat: point.lat(),
            lng: point.lng(),
          }));
        }
      } else {
        console.error('Error al calcular la ruta: ', status);
      }
    }
  );
};

const onClickPolimarket = (e: google.maps.MapMouseEvent) => {
  const latLng = e.latLng;
  if (latLng) {
    const lat = latLng.lat();
    const lng = latLng.lng();
    alert(`${lat}, ${lng}`);
  }
};

// onMounted(() => {
//   if (mapRef.value) {
//     console.log('SI EXISTE ->', mapRef.value);
//   }
//   getCurrentLocation();
// });
onMounted(async () => {
  // Request Google Maps Libraries
  const { Map } = (await google.maps.importLibrary(
    'maps'
  )) as google.maps.MapsLibrary;

  // Initialize the Map
  mapRef.value = new Map(document.getElementById('map') as HTMLElement, {
    zoom: 4,
    center: center.value,
  });

  // Setup the DirectionsRenderer
  directionsRenderer.value = new google.maps.DirectionsRenderer();
  directionsRenderer.value.setMap(mapRef.value);

  // Add map click listener to add markers
  mapRef.value.addListener('click', addMarker);

  // Get user's current location
  getCurrentLocation();
});
</script>

<style lang="scss" scoped>
.map {
  position: relative;
  width: 100%;
  height: 500px;
}

.map::after {
  position: absolute;
  content: '';
  width: 1px;
  height: 100%;
  top: 0;
  left: 50%;
  background: red;
}

label {
  font-weight: 500;
}

input[type='number'] {
  margin-top: 20px;
  margin-left: 10px;
  outline: 1px solid #ccc;
  border-radius: 4px;
}
</style>
