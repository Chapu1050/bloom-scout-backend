<script setup lang="ts">
import { formatDate } from "@/utils/formatDate";

const props = defineProps(["observation"]);
</script>

<template>
  <h1>{{ observation.title }}</h1>
  <p class="author">{{ observation.author }}</p>
  <p>{{ observation.description }}</p>
  <img v-if="observation.imageUrl" :src="observation.imageUrl" alt="Observation Image" class="observation-image" />
  <div class="timestamp">
    <p>Created on: {{ formatDate(observation.dateCreated) }}</p>
  </div>
</template>

<style scoped>
p {
  margin: 0em;
}

.author {
  font-weight: bold;
  font-size: 1.2em;
}

.timestamp {
  font-size: 0.9em;
  font-style: italic;
  margin-top: 0.5em;
}

.observation-image {
  width: 100%;
  height: auto;
  border-radius: 8px;
  margin-top: 0.5em;
}
</style>
