<template>
  <GoogleMap :api-key="apiKey" :options="mapOptions">
    <GMapMarker
      v-for="marker in markers"
      :key="marker.id"
      :position="marker.position"
      @click="toggleMarker(marker)"
    />
    <GMapPolyline :path="polylinePath" />
  </GoogleMap>
  <p>Distancia total: {{ totalDistance }} metros</p>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue';
import GoogleMap from 'vue3-google-map';
import GMapMarker from 'vue3-google-map';
import GMapPolyline from 'vue3-google-map';

interface Marker {
  id: number;
  position: google.maps.LatLngLiteral;
}

const apiKey = 'AIzaSyBhkuXoZ4HxXbSOcGUzZAvdRgmzWJbzxbA'; // Reemplaza con tu clave

const markers = ref<Marker[]>([]);
const polylinePath = ref<google.maps.LatLngLiteral[]>([]);
const totalDistance = ref<number | null>(null);

const mapOptions = ref({
  center: { lat: -34.397, lng: 150.644 },
  zoom: 8,
});

const toggleMarker = (marker: Marker) => {
  const index = polylinePath.value.indexOf(marker.position);
  if (index > -1) {
    polylinePath.value.splice(index, 1);
  } else {
    polylinePath.value.push(marker.position);
  }
};

const calculateDistance = () => {
  if (polylinePath.value.length < 2) return 0;

  const service = new window.google.maps.DistanceMatrixService();
  service.getDistanceMatrix(
    {
      origins: polylinePath.value,
      destinations: polylinePath.value,
      travelMode: google.maps.TravelMode.DRIVING, // Puedes cambiar el modo de transporte
    },
    (response, status) => {
      if (status === 'OK') {
        const origins = response.originAddresses;
        const destinations = response.destinationAddresses;
        const distances = response.rows[0].elements;
        let total = 0;
        for (let i = 0; i < distances.length; i++) {
          total += distances[i].distance.value;
        }
        totalDistance.value = total;
      }
    }
  );
};

watch(polylinePath, calculateDistance);
</script>
