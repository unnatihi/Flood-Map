<template>
    <div class="search-bar">
      <input
        v-model="searchQuery"
        type="text"
        placeholder="Search for a place"
        @input="fetchSuggestions"
      />
      <ul v-if="suggestions.length" class="suggestions">
        <li
          v-for="(suggestion, index) in suggestions"
          :key="index"
          @click="selectSuggestion(suggestion)"
        >
          {{ suggestion.display_name }}
        </li>
      </ul>
    </div>
  </template>
  
  <script setup lang="ts">
  import { ref, defineEmits } from 'vue';
  import axios from 'axios';
  
  // Emits coordinates and selected place name to the parent
  const emit = defineEmits<{
    (e: 'place-selected', coordinates: [number, number], name: string): void;
  }>();
  
  const searchQuery = ref<string>('');
  const suggestions = ref<any[]>([]);
  
  async function fetchSuggestions() {
    if (!searchQuery.value.trim()) {
      suggestions.value = [];
      return;
    }
  
    try {
      const response = await axios.get('https://nominatim.openstreetmap.org/search', {
        params: {
          q: searchQuery.value,
          format: 'json',
          addressdetails: 1,
          limit: 5, // Limit the number of suggestions
        },
      });
  
      suggestions.value = response.data;
    } catch (error) {
      console.error('Error fetching suggestions:', error);
      suggestions.value = [];
    }
  }
  
  function selectSuggestion(suggestion: any) {
    const coordinates: [number, number] = [
      parseFloat(suggestion.lon),
      parseFloat(suggestion.lat),
    ];
    emit('place-selected', coordinates, suggestion.display_name);
    searchQuery.value = suggestion.display_name;
    suggestions.value = []; // Clear suggestions after selection
  }
  </script>
  
  <style scoped>
  .search-bar {
    position: relative;
  }
  
  input {
    padding: 5px;
    font-size: 14px;
    width: 100%;
  }
  
  .suggestions {
    position: absolute;
    top: 100%;
    left: 0;
    right: 0;
    background: white;
    border: 1px solid #ccc;
    max-height: 200px;
    overflow-y: auto;
    list-style: none;
    margin: 0;
    padding: 0;
    z-index: 1000;
  }
  
  .suggestions li {
    padding: 8px 12px;
    cursor: pointer;
  }
  
  .suggestions li:hover {
    background: #f0f0f0;
  }
  </style>
  