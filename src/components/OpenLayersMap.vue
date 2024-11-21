<template>
  <div>
    <!-- Map Top Bar -->
    <div class="map-top-bar">
      <!-- Search Component -->
      <SearchComponent @place-selected="addMarker" />

      <!-- Layer Toggle Icon -->
      <button class="layer-icon" @click="toggleLayerMenu">
        <img
          src="https://cdn-icons-png.flaticon.com/512/786/786432.png"
          alt="Layers"
        />
      </button>

      <!-- Clear Button -->
      <button @click="clearMarker" class="clear-button">Clear</button>
    </div>

    <!-- Layer Controls -->
    <div v-if="showLayerMenu" class="layer-controls">
      <label>
        <input
          type="checkbox"
          v-model="showNorthCarolina"
          @change="toggleLayer('NorthCarolina')"
        />
        North Carolina
      </label>
      <label>
        <input
          type="checkbox"
          v-model="showPlace"
          @change="toggleLayer('Place')"
        />
        USA
      </label>
      <label>
        <input
          type="checkbox"
          v-model="showFlood"
          @change="toggleLayer('Flood')"
        />
        Flood Areas
      </label>
    </div>

    <!-- Map Container -->
    <div ref="mapContainer" class="map-container"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import 'ol/ol.css';
import { Map, View } from 'ol';
import TileLayer from 'ol/layer/Tile';
import VectorLayer from 'ol/layer/Vector';
import OSM from 'ol/source/OSM';
import VectorSource from 'ol/source/Vector';
import GeoJSON from 'ol/format/GeoJSON';
import { fromLonLat } from 'ol/proj';
import Feature from 'ol/Feature';
import Point from 'ol/geom/Point';
import { Fill, Stroke, Style, Icon } from 'ol/style';
import { defaults as defaultControls, Zoom } from 'ol/control';
import SearchComponent from './SearchComponent.vue';

// Map references and states
const mapContainer = ref<HTMLDivElement | null>(null);
const map = ref<Map | null>(null);
const northCarolinaLayer = ref<VectorLayer<VectorSource> | null>(null);
const placeLayer = ref<VectorLayer<VectorSource> | null>(null);
const floodLayer = ref<VectorLayer<VectorSource> | null>(null);
const markerLayer = ref<VectorLayer<VectorSource> | null>(null);

// Layer visibility states
const showNorthCarolina = ref(false);
const showPlace = ref(false);
const showFlood = ref(false);
const showLayerMenu = ref(false); // For toggling layer menu visibility

const initialView = {
  center: fromLonLat([-98.5795, 39.8283]),
  zoom: 4,
};

const floodAreaExtent = [-10000000, 4000000, -9000000, 5000000];

function initMap(): void {
  if (mapContainer.value) {
    map.value = new Map({
      target: mapContainer.value,
      layers: [
        new TileLayer({
          source: new OSM(),
        }),
      ],
      view: new View({
        center: initialView.center,
        zoom: initialView.zoom,
        projection: 'EPSG:3857',
      }),
      controls: defaultControls().extend([
        new Zoom({
          className: 'custom-zoom', // Custom zoom control style
        }),
      ]),
    });
  }
}

function toggleLayer(layer: string): void {
  if (!map.value) return;

  if (layer === 'NorthCarolina') {
    if (!northCarolinaLayer.value) {
      northCarolinaLayer.value = new VectorLayer({
        source: new VectorSource({
          url: '/northcarolina.geojson',
          format: new GeoJSON(),
        }),
      });
      map.value.addLayer(northCarolinaLayer.value);
    }
    northCarolinaLayer.value.setVisible(showNorthCarolina.value);
  } else if (layer === 'Place') {
    if (!placeLayer.value) {
      placeLayer.value = new VectorLayer({
        source: new VectorSource({
          url: '/place.geojson',
          format: new GeoJSON(),
        }),
      });
      map.value.addLayer(placeLayer.value);
    }
    placeLayer.value.setVisible(showPlace.value);
  } else if (layer === 'Flood') {
    if (!floodLayer.value) {
      floodLayer.value = new VectorLayer({
        source: new VectorSource({
          url: '/flood.geojson',
          format: new GeoJSON(),
        }),
        style: new Style({
          fill: new Fill({
            color: 'rgba(0, 0, 255, 0.2)',
          }),
          stroke: new Stroke({
            color: 'blue',
            width: 2,
          }),
        }),
      });
      map.value.addLayer(floodLayer.value);
    }
    floodLayer.value.setVisible(showFlood.value);

    if (showFlood.value) {
      map.value.getView().fit(floodAreaExtent, { size: map.value.getSize(), maxZoom: 10 });
    } else {
      map.value.getView().setCenter(initialView.center);
      map.value.getView().setZoom(initialView.zoom);
    }
  }
}

function toggleLayerMenu(): void {
  showLayerMenu.value = !showLayerMenu.value;
}

function addMarker(coordinates: [number, number], name: string): void {
  if (!map.value) return;

  if (!markerLayer.value) {
    markerLayer.value = new VectorLayer({
      source: new VectorSource(),
    });
    map.value.addLayer(markerLayer.value);
  }

  const marker = new Feature({
    geometry: new Point(fromLonLat(coordinates)),
    name,
  });

  marker.setStyle(
    new Style({
      image: new Icon({
        src: 'https://cdn-icons-png.flaticon.com/512/684/684908.png',
        scale: 0.05,
      }),
    })
  );

  markerLayer.value.getSource()?.addFeature(marker);
  map.value.getView().setCenter(fromLonLat(coordinates));
  map.value.getView().setZoom(10);
}

function clearMarker(): void {
  markerLayer.value?.getSource()?.clear();
}

onMounted(initMap);
</script>

<style scoped>
.map-container {
  width: 100%;
  height: 100%;
  position: absolute;
}

.map-top-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: rgba(255, 255, 255, 0.8);
  padding: 5px 10px;
  position: absolute;
  top: 10px;
  left: 10px;
  right: 10px;
  z-index: 1000;
  border-radius: 8px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
}

.layer-icon img {
  width: 24px;
  height: 24px;
  cursor: pointer;
}

.clear-button {
  background-color: #f44336;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 5px 10px;
  cursor: pointer;
}

.clear-button:hover {
  background-color: #d32f2f;
}

.layer-controls {
  position: absolute;
  top: 60px;
  left: 10px;
  background: rgba(255, 255, 255, 0.9);
  padding: 10px;
  border-radius: 8px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
  z-index: 1000;
}

/* Custom Zoom Controls */
.custom-zoom {
  position: absolute;
  top: 70px; /* Adjusted to be below the navbar */
  right: 10px;
  z-index: 1000;
}
</style>
