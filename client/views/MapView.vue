<template>
  <div class="map-container">
    <div class="sidebar">
      <button @click="toggleObservations" class="toggle-button">
        {{ showUserObservations ? 'Show All Observations' : 'Show My Observations' }}
      </button>
    </div>
    <div id="map" class="google-map"></div> <!-- Container for Google Map -->
  </div>
</template>

<reference types="vite/client" />


<script setup lang="ts">
import { onMounted, ref } from 'vue';
interface Observation {
  location: {
    latitude: number;
    longitude: number;
  };
  title: string;
  content: string;
  userId: string;
  imageUrl: string;
  username?: string; // Optional, as it will be added later
}

const center = { lat: 42.3601, lng: -71.0942 }; // Default center
const allObservations = ref<Observation[]>([]); // All observations
const userObservations = ref<Observation[]>([]); // User's observations
const filteredObservations = ref<Observation[]>([]); // Currently displayed observations
const showUserObservations = ref(false); // Toggle for user observations
let map: google.maps.Map | undefined; // Declare map variable with type
let markers: google.maps.Marker[] = []; // Array to hold markers

// Initialize Google Map
const initMap = () => {
  map = new google.maps.Map(document.getElementById("map") as HTMLElement, {
    center: center,
    zoom: 15,
    disableDefaultUI: true,
    gestureHandling: 'greedy',
  });
};

// Clear existing markers from the map
const clearMarkers = () => {
  markers.forEach(marker => marker.setMap(null)); // Remove marker from the map
  markers = []; // Clear the markers array
};

// Add markers based on filtered observations
const addMarkers = () => {
  clearMarkers(); // Clear existing markers
  filteredObservations.value.forEach((observation) => {
    const marker = new google.maps.Marker({
      position: { lat: observation.location.latitude, lng: observation.location.longitude },
      map: map,
      title: observation.title,
    });

    markers.push(marker); // Add marker to markers array

    // InfoWindow to show on marker click
    const infoWindow = new google.maps.InfoWindow({
      content: `
        <div class="info-window-content">
          <h3>${observation.title}</h3>
          <p>${observation.content}</p>
          <p><strong>Posted by:</strong> ${observation.username || 'Unknown'}</p>
          <img src="${observation.imageUrl}" alt="Observation Image" 
               style="width: 150px; height: 100px; margin-top: 0.5em; border-radius: 8px; object-fit: cover;" />
        </div>
      `,
    });

    marker.addListener("click", () => {
      infoWindow.open(map, marker);
    });
  });
};

// Fetch all observations from your API
const fetchObservations = async () => {
  try {
    const response = await fetch('/api/observations'); // Update with your actual API endpoint
    if (!response.ok) {
      throw new Error('Failed to fetch observations');
    }
    allObservations.value = await response.json();
    await addUsernameToObservations(allObservations.value); // Add usernames to observations
    filteredObservations.value = allObservations.value; // Initialize filtered observations
    addMarkers(); // Add markers after fetching observations
  } catch (error) {
    console.error(error); // Handle the error as needed
  }
};

// Fetch username from user ID
const fetchUsernameFromId = async (id: string): Promise<string> => {
  try {
    const response = await fetch(`/api/username/${id}`); // Update with your actual API endpoint
    if (!response.ok) {
      throw new Error(`Failed to fetch user with ID ${id}`);
    }
    const userData = await response.json();
    return userData.username;
  } catch (error) {
    console.error(error); // Handle the error as needed
    return 'Unknown'; // Fallback if user not found
  }
};

// Fetch user observations from your API
const fetchUserObservations = async () => {
  try {
    const response = await fetch('/api/observations/user'); 
    if (!response.ok) {
      throw new Error('Failed to fetch user observations');
    }
    userObservations.value = await response.json();
    await addUsernameToObservations(userObservations.value); // Add usernames to user observations
  } catch (error) {
    console.error(error); // Handle the error as needed
  }
};

// Fetch username for each observation and add it to the observation
const addUsernameToObservations = async (observations: Observation[]) => {
  for (const observation of observations) {
    observation.username = await fetchUsernameFromId(observation.userId);
  }
};

// Toggle observation filter
const toggleObservations = () => {
  showUserObservations.value = !showUserObservations.value;
  filteredObservations.value = showUserObservations.value ? userObservations.value : allObservations.value;

  // Re-initialize map markers when toggling
  addMarkers();
};

// Load Google Maps script and initialize the map
onMounted(async () => {
  // Load Google Maps JavaScript API
  const script = document.createElement('script');
  script.src = `https://maps.googleapis.com/maps/api/js?key=AIzaSyAPXmxyv4aAVoGAksTMoqkF-YxTqWcBalw&callback=initMap`;
  script.async = true;
  script.defer = true;
  document.head.appendChild(script);
  script.onload = async () => {
    initMap();
    await fetchObservations();
    await fetchUserObservations();
  };
});
</script>

<style scoped>
.toggle-button {
  position: absolute;
  z-index: 1;
  right: 8%; /* Distance from the right */
  top: 16%; /* Distance from the top */
  background-color: #A7C957; /* Button background color */
  color: #ffffff; /* Button text color */
  border: none; /* Remove default border */
  border-radius: 5px; /* Rounded corners */
  padding: 10px 15px; /* Padding for the button */
  font-size: 16px; /* Font size */
  cursor: pointer; /* Change cursor on hover */
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); /* Shadow effect */
  transition: background-color 0.3s, transform 0.3s; /* Transition effects */
}

.toggle-button:hover {
  background-color: #8FAE47; /* Darker shade on hover */
  transform: translateY(-2px); /* Slight lift on hover */
}

.map-container {
  position: absolute;
  right: 0;
  bottom: 0;
  height: 100%;
  width: 100%;
  z-index: 0;
}

.google-map {
  height: 100%;
  width: 100%;
}

.info-window-content {
  max-width: 200px;
}
</style>
