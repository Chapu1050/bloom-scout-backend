<script setup>
import { onMounted, ref } from 'vue';
import { GoogleMap, InfoWindow, Marker } from 'vue3-google-map';

const center = { lat: 42.3601, lng: -71.0942 }; // Default center
const apiKey = "AIzaSyAPXmxyv4aAVoGAksTMoqkF-YxTqWcBalw"; // Your API key

// Observations data
const observations = ref([]); // Start with an empty array



const mapOptions = {
  disableDefaultUI: true,
  zoomControl: false,
  streetViewControl: false,
  mapTypeControl: false,
  fullscreenControl: false,
  gestureHandling: 'greedy',
  styles: [
    {
      featureType: "poi",
      elementType: "labels",
      stylers: [{ visibility: "off" }]
    },
    {
      featureType: "transit",
      elementType: "labels",
      stylers: [{ visibility: "off" }]
    }
  ]
};

// Fetch observations from your API
const fetchObservations = async () => {
  try {
    const response = await fetch('/api/observations'); // Update with your actual API endpoint
    if (!response.ok) {
      throw new Error('Failed to fetch observations');
    }
    observations.value = await response.json();
    console.log(observations.value);
  } catch (error) {
    console.error(error); // Handle the error as needed
  }
};



onMounted(fetchObservations);
</script>

<template>
  <div class="map-container">
    <GoogleMap
      :api-key="apiKey"
      style="width: 100%; height: 100%"
      :center="center"
      :zoom="15"
      :options="mapOptions"
    >
      <Marker
        v-for="observation in observations"
        :key="observation.id"
        :options="{ position: { lat: observation.location.latitude, lng: observation.location.longitude } }"
        :title="observation.title"
      >
        <!-- Show InfoWindow only if it is the selected observation -->
        <InfoWindow>
          <div class="info-window-content">
            <h3>{{ observation.title }}</h3>
            <p>{{ observation.content }}</p>
            <img v-if="observation.imageUrl" :src="observation.imageUrl" alt="Observation Image" class="observation-image" />
          </div>
        </InfoWindow>
      </Marker>
    </GoogleMap>
  </div>
</template>

<style scoped>
.map-container {
  position: absolute;
  right: 0;
  bottom: 0;
  height: 100%;
  width: 100%;
  z-index: 0;
}

.info-window-content {
  max-width: 200px;
}

.observation-image {
  width: 100%;
  height: auto;
  margin-top: 0.5em;
  border-radius: 8px;
}
</style>
