<template>
  <div class="globe-wrapper">
    <div ref="globeContainer" class="globe-container"></div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import * as THREE from 'three';
import Globe from 'globe.gl';

const globeContainer = ref(null);
let globeInstance = null;

// Configuration constants
const GLOBE_CONFIG = {
  cities: [
    { lat: 39.0437567, lng: -77.4874416, name: "Ashburn, Virginia (US)"},
    { lat: 51.509865,  lng: -0.118092,   name: "London, United Kingdom"},
    { lat: 20.659698,  lng: -103.349609, name: "Guadalajara, Mexico"},
    { lat: 52.237049,  lng: 21.017532,   name: "Warsaw, Poland"},
    { lat: 34.052235, lng: -118.243683, name: "Los Angeles, California (US)"},
    { lat: 52.371807, lng: 4.896029, name: "Amsterdam, Netherlands"},
    { lat: 35.658581, lng: 139.745438, name: "Tokyo, Japan"},
    { lat: -23.533773, lng: -46.625290, name: "Sao Paulo, Brazil"},
    { lat: 44.439663, lng: 26.096306, name: "Bucharest, Romania"},
    { lat: 22.302711, lng: 114.177216, name: "Hong Kong, Hong Kong"},
    { lat: 1.290270, lng: 103.851959, name: "Singapore, Singapore"},
  ],
  baseCameraDistance: 300,
  arcColor: 'orange',
  pointColor: 'orange',
  pointRadius: 1.0,
  arcDashLength: 0.1,
  arcDashGap: 0.1,
  arcDashAnimateTime: 3500,
  earthTexture: '/earth-night.jpg'
};

function initGlobe() {
  if (!globeContainer.value) return;

  // Clean up existing instance if it exists
  if (globeInstance) {
    globeInstance._destructor();
    globeInstance = null;
  }

  const { cities, baseCameraDistance, arcColor, pointColor, pointRadius, arcDashLength, arcDashGap, arcDashAnimateTime, earthTexture } = GLOBE_CONFIG;

  // Generate arcs data
  const arcsData = cities.slice(0, -1).map((startCity, i) => ({
    startLat: startCity.lat,
    startLng: startCity.lng,
    endLat: cities[i + 1].lat,
    endLng: cities[i + 1].lng,
    color: arcColor,
  }));

  // Process cities data
  const processedCities = cities.map(city => ({
    ...city,
    lat: +city.lat,
    lng: +city.lng,
    altitude: 0,
    text: city.name
  }));

  globeInstance = Globe({
    rendererConfig: {
      alpha: true,
      useLegacyLights: false,
      antialias: true,
      powerPreference: 'high-performance',
      precision: 'highp',
      preserveDrawingBuffer: true,
      stencil: false,
      depth: true
    },
    controlType: 'orbit',
    enableZoom: false,
    waitForGlobeReady: true,
    animateIn: false
  })(globeContainer.value)
      .width(globeContainer.value.clientWidth)
      .height(globeContainer.value.clientHeight)
      .globeImageUrl(earthTexture)
      .backgroundColor('rgba(0,0,0,0)')
      .arcsData(arcsData)
      .arcColor('color')
      .arcDashLength(arcDashLength)
      .arcDashGap(arcDashGap)
      .arcDashAnimateTime(arcDashAnimateTime)
      .pointsData(processedCities)
      .pointColor(() => pointColor)
      .pointRadius(pointRadius)
      .pointAltitude(0)
      .pointLabel(d => d.name)
      .atmosphereColor('#ffffff')
      .atmosphereAltitude(0.1)
      .lights([
        new THREE.AmbientLight(0xffffff, 1.0),
        new THREE.DirectionalLight(0xffffff, 1.0)
      ]);

  // Function to calculate appropriate camera distance based on container size
  const calculateCameraDistance = () => {
    const containerWidth = globeContainer.value.clientWidth;
    const scaleFactors = {
      small: 1.2,  // < 600px
      medium: 1.1, // < 900px
      large: 1.0   // >= 900px
    };

    const scaleFactor = containerWidth < 600 ? scaleFactors.small :
        containerWidth < 900 ? scaleFactors.medium :
            scaleFactors.large;

    return baseCameraDistance * scaleFactor;
  };

  // Function to update camera position
  const updateCameraPosition = () => {
    if (!globeInstance) return;
    const distance = calculateCameraDistance();
    globeInstance.controls().minDistance = distance;
    globeInstance.controls().maxDistance = distance;
    globeInstance.camera().position.z = distance;
  };

  // Initial camera position
  updateCameraPosition();

  // Handle resize
  const handleResize = () => {
    if (!globeInstance || !globeContainer.value) return;
    globeInstance
        .width(globeContainer.value.clientWidth)
        .height(globeContainer.value.clientHeight);
    updateCameraPosition();
  };

  window.addEventListener('resize', handleResize);
}

// Cleanup function
const cleanup = () => {
  if (globeInstance) {
    globeInstance._destructor();
    globeInstance = null;
  }
};

onMounted(() => {
  if (typeof window !== 'undefined') {
    initGlobe();
  }
});

onUnmounted(() => {
  cleanup();
});
</script>

<style scoped>
.globe-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 800px;
}

.globe-container {
  width: 800px;
  height: 800px;
  position: relative;
  margin: 0 auto;
}

@media (max-width: 1280px) {
  .globe-wrapper {
    height: 600px;
  }

  .globe-container {
    width: 600px;
    height: 600px;
  }
}

@media (max-width: 900px) {
  .globe-wrapper {
    height: 500px;
  }

  .globe-container {
    width: 500px;
    height: 500px;
  }
}

@media (max-width: 640px) {
  .globe-wrapper {
    height: 400px;
  }

  .globe-container {
    width: 400px;
    height: 400px;
  }
}
</style>