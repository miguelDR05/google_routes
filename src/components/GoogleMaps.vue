<template>
  <GoogleMap
    :api-key="googleMapsApiKey"
    style="width: 100%; height: 500px"
    :center="center"
    :zoom="zoom"
    ref="mapRef"
    @click="addMarker"
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
    <Polyline :options="flightPath" @click="onClick" />
    <Marker
      v-for="(marker, index) in markers"
      :key="index"
      :options="{ position: marker }"
      @click="selectMarker(index)"
    />
  </GoogleMap>
  <q-btn @click="calculateRoute" :disabled="markers.length < 2">
    Calcular Ruta
  </q-btn>
  <q-btn @click="removeSelectedMarker" :disabled="selectedMarkerIndex === null">
    Eliminar Marcador
  </q-btn>
  {{ flightPath.path }}
</template>

<script setup lang="ts">
/* eslint-disable no-undef */
import { ref, onMounted } from 'vue';
import { GoogleMap, Polyline, Marker } from 'vue3-google-map'; // InfoWindow
const googleMapsApiKey = 'AIzaSyBhkuXoZ4HxXbSOcGUzZAvdRgmzWJbzxbA';

interface Coordinates {
  lat: number;
  lng: number;
}

// const center = ref<Coordinates>({ lat: 37.739, lng: -111.1 });
// const center = ref<Coordinates>({ lat: 0, lng: -122 });
const center = ref<Coordinates>({ lat: 37.772, lng: -122.214 }); // Centro inicial
const zoom = ref(10); // Nivel de zoom
const markers = ref<Coordinates[]>([]);
// const mapRef = ref<InstanceType<typeof GoogleMap> | null>(null);
const mapRef = ref<google.maps.Map | null>(null);
// const locations = ref<Array<Coordinates>>([
//   { lat: 37.75, lng: -111.116667 },
//   { lat: 37.759859, lng: -111.128708 },
//   { lat: 37.765015, lng: -111.133858 },
//   { lat: 37.770104, lng: -111.143299 },
//   { lat: 37.7737, lng: -111 - 111187 },
//   { lat: 37.774785, lng: -111.137978 },
//   { lat: 37.819616, lng: 144.968119 },
//   { lat: 38.330766, lng: 144.695692 },
//   { lat: 39.927193, lng: 175.053218 },
//   { lat: 41.330162, lng: 174.865694 },
//   { lat: 42.734358, lng: 147.439506 },
//   { lat: 42.734358, lng: 147.501315 },
//   { lat: 42.735258, lng: 147.438 },
//   { lat: 43.999792, lng: 170.463352 },
// ]);

// const flightPlanCoordinates = ref<Array<Coordinates>>([
//   { lat: 37.772, lng: -122.214 },
//   { lat: 21.291, lng: -157.821 },
//   { lat: -18.142, lng: 178.431 },
//   { lat: -27.467, lng: 153.027 },
// ]);

const flightPath = ref({
  path: [] as Coordinates[],
  geodesic: true,
  strokeColor: '#FF0000',
  strokeOpacity: 1.0,
  strokeWeight: 2,
});

const addMarker = (event: google.maps.MapMouseEvent) => {
  console.log('click', event);

  if (event.latLng) {
    console.log('lat', event.latLng.lat());
    console.log('lng', event.latLng.lng());

    markers.value.push({
      lat: event.latLng.lat(),
      lng: event.latLng.lng(),
    });
  }
};
const selectedMarkerIndex = ref<number | null>(null);
/*
const calculateRoute = () => {
  if (markers.value.length < 2) {
    console.error('Necesitas al menos 2 puntos para calcular la ruta.');
    return;
  }

  const directionsService = new google.maps.DirectionsService();
  const directionsRenderer = new google.maps.DirectionsRenderer();

  if (mapRef.value) {
    directionsRenderer.setMap(mapRef.value);
  } else {
    console.error('El mapa no está disponible.');
    return;
  }

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
      travelMode: google.maps.TravelMode.DRIVING,
    },
    (response, status) => {
      if (status === google.maps.DirectionsStatus.OK && response) {
        directionsRenderer.setDirections(response);

        const route = response.routes[0];
        const distance = route.legs.reduce(
          (acc, leg) => acc + (leg.distance?.value ?? 0),
          0
        );
        console.log(`Distancia total: ${distance / 1000} km`);

        // Actualizar el path de Polyline
        flightPath.value.path = markers.value;
      } else {
        console.error('Error al calcular la ruta: ', status);
      }
    }
  );
};
*/

const selectMarker = (index: number) => {
  selectedMarkerIndex.value = index;
};

const removeSelectedMarker = () => {
  if (selectedMarkerIndex.value !== null) {
    markers.value.splice(selectedMarkerIndex.value, 1);
    selectedMarkerIndex.value = null;
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
  const directionsRenderer = new google.maps.DirectionsRenderer();

  directionsRenderer.setMap(mapRef.value);

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
        directionsRenderer.setDirections(response);

        const route = response.routes[0];
        const distance = route.legs.reduce(
          (acc, leg) => acc + (leg.distance?.value ?? 0),
          0
        );
        console.log(`Distancia total: ${distance / 1000} km`);

        // Actualiza el vueloPath solo si hay una ruta
        if (route) {
          console.log('actualizando lineas');

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
const onClick = (e) => {
  const { lat, lng } = e.latLng;
  alert(`${lat()}, ${lng()}`);
};

// Asegúrate de que la API se haya cargado antes de intentar usarla
onMounted(() => {
  // nextTick(() => {
  //   if (mapRef.value) {
  //     mapRef.value = mapRef.value.$mapObject;
  //   }
  // });
});
</script>

<style></style>
