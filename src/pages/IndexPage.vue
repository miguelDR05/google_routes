<template>
  <q-page class="row items-center justify-evenly">
    <example-component
      v-if="false"
      title="COMPONENTE DE EJEMPLO"
      active
      :todos="todos"
      :meta="meta"
    ></example-component>
    <google-mapsv2></google-mapsv2>
    <!-- <maps-route></maps-route> -->
  </q-page>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { Todo, Meta } from 'components/models';
import ExampleComponent from 'components/ExampleComponent.vue';
import GoogleMapsv2 from 'components/GoogleMapsv2.vue';
// import GoogleMaps from 'components/GoogleMaps.vue';
// import MapsRoute from 'components/MapsRoute.vue';

defineOptions({
  name: 'IndexPage',
});

const todos = ref<Todo[]>([
  {
    id: 1,
    content: 'ct1',
  },
  {
    id: 2,
    content: 'ct2',
  },
  {
    id: 3,
    content: 'ct3',
  },
  {
    id: 4,
    content: 'ct4',
  },
  {
    id: 5,
    content: 'ct5',
  },
]);

const meta = ref<Meta>({
  totalCount: 1200,
});
</script>
