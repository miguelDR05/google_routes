<template>
  <div>
    <div id="map" class="map"></div>
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
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';

interface Coordinates {
  lat: number;
  lng: number;
}

// const googleMapsApiKey = 'YOUR_GOOGLE_MAPS_API_KEY';
const center = ref<Coordinates>({ lat: 37.772, lng: -122.214 });
const markers = ref<Coordinates[]>([]);
const mapRef = ref<google.maps.Map | null>(null);
const directionsRenderer = ref<google.maps.DirectionsRenderer | null>(null);
const selectedMarkerIndex = ref<number | null>(null);

const flightPath = ref({
  path: [] as Coordinates[],
  geodesic: true,
  strokeColor: '#FF0000',
  strokeOpacity: 1.0,
  strokeWeight: 2,
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
  if (mapRef.value) {
    flightPath.value.path = markers.value;
    const polyline = new google.maps.Polyline(flightPath.value);
    polyline.setMap(mapRef.value);
  }
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
      markers.value.push({
        lat: position.coords.latitude,
        lng: position.coords.longitude,
      });
      updateFlightPath();
    });
  } else {
    console.log('Geolocation is not supported by this browser.');
  }
};

const calculateRoute = () => {
  if (markers.value.length < 2) {
    console.error('At least 2 points are required to calculate a route');
    return;
  }

  const directionsService = new google.maps.DirectionsService();

  directionsService.route(
    {
      origin: markers.value[0],
      destination: markers.value[markers.value.length - 1],
      waypoints: markers.value.slice(1, -1).map((location) => ({ location })),
      optimizeWaypoints: true,
      travelMode: google.maps.TravelMode.DRIVING,
    },
    (response, status) => {
      if (status === google.maps.DirectionsStatus.OK && response) {
        directionsRenderer.value?.setDirections(response);

        const route = response.routes[0];
        if (route) {
          flightPath.value.path = route.overview_path.map((point) => ({
            lat: point.lat(),
            lng: point.lng(),
          }));
          updateFlightPath();
        }
      } else {
        console.error('Error calculating route:', status);
      }
    }
  );
};

// Initialize the map and setup the event listeners
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

<style scoped>
.map {
  width: 100%;
  height: 500px;
}
</style>
