<template>
    <div ref="canvasContainer" class="h-screen w-screen">
        <div class="absolute inset-0  flex justify-center items-center" style="pointer-events: none;">
                <button style="pointer-events: auto;" class="h-10 px-3 w-fit bg-slate-200 rounded-lg"  @click="leave">START</button>
              </div>
    </div>


    <p>Loaded Images: {{ loadedImagesCount }}</p>

    <!-- 
    <div class="h-screen w-screen">
        <div ref='threejsMap'>
            <div class="absolute inset-0  flex justify-center items-center" style="pointer-events: none;">
                <button style="pointer-events: auto;" class="h-10 px-3 w-fit bg-slate-200 rounded-lg"  @click="leave">START</button>
            </div>
        </div>
    </div>
    -->
</template>
  
<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue';
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'

const emit = defineEmits(['leave'])
const leave = () => {
    emit('leave')
}

const loadedImagesCount = ref(0);

const canvasContainer = ref<HTMLElement | null>(null);
let scene: THREE.Scene, camera: THREE.PerspectiveCamera, renderer: THREE.WebGLRenderer;
let controls: OrbitControls;

const initThree = () => {
scene = new THREE.Scene();
const fov = 75;
const aspect = window.innerWidth / window.innerHeight;
const near = 0.1;
const far = 1000;
camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
camera.position.z = 5;

renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
if (canvasContainer.value) {
    canvasContainer.value.appendChild(renderer.domElement);
}

controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;

const light = new THREE.AmbientLight(0xffffff, 0.5);
scene.add(light);
};


interface ImageData {
  url: string;
  width: number;
  height: number;
  x: number;
  y: number;
  z: number;
}

// Define a constant multiplier
const Y_COORD_MULTIPLIER = 80000; // Example large number

const loadImagesFromCSV = async (csvPath: string, n: number) => {
  const response = await fetch(csvPath);
  const csvText = await response.text();
  const lines = csvText.split('\n');

  // Identify the column indexes
  const headers = lines[0].split(',').map(header => header.trim());
  const filenameIndex = headers.indexOf('image_filename');
  const widthIndex = headers.indexOf('image_width');
  const heightIndex = headers.indexOf('image_height');
  const xCoordIndex = headers.indexOf('tsne_x'); // Index for x coordinate
  const yCoordIndex = headers.indexOf('density_3'); // Index for y coordinate
  const zCoordIndex = headers.indexOf('tsne_y'); // Index for z coordinate

  if (filenameIndex === -1 || widthIndex === -1 || heightIndex === -1 || xCoordIndex === -1 || yCoordIndex === -1 || zCoordIndex === -1) {
    console.error('Required column(s) not found');
    return [];
  }

  const basePath = '/sample_images_10k_orig01/';

  // Extract image filenames, widths, heights, and coordinates
  return lines.slice(1, n + 1)
    .map(line => {
      const columns = line.split(',').map(column => column.trim());
      const filename = columns[filenameIndex];
      const width = parseFloat(columns[widthIndex]);
      const height = parseFloat(columns[heightIndex]);
      const xCoord = parseFloat(columns[xCoordIndex]); // Extract x coordinate
      const yCoord = parseFloat(columns[yCoordIndex]) * Y_COORD_MULTIPLIER; // Apply multiplier to y coordinate
      const zCoord = parseFloat(columns[zCoordIndex]); // Extract z coordinate
      return { url: basePath + filename, width, height, x: xCoord, y: yCoord, z: zCoord };
    })
    .filter(data => data.url !== basePath);
};

const loadImagesToGallery = async (imageData: ImageData[]) => {
  for (const { url, width, height, x, y, z } of imageData) {
    const aspectRatio = width / height;
    const geometry = new THREE.PlaneGeometry(aspectRatio, 1); // Use aspect ratio to size geometry
    const texture = await new Promise<THREE.Texture>((resolve, reject) => {
      new THREE.TextureLoader().load(url, resolve, undefined, reject);
      loadedImagesCount.value++; // For Composition API
    });
    const material = new THREE.MeshBasicMaterial({ map: texture });
    const plane = new THREE.Mesh(geometry, material);
    // Set the position of the plane based on x, y, z coordinates
    plane.position.set(x, y, z);
    scene.add(plane);
  }
};

async function initGallery() {
  const csvPath = '/tsne_coordinates_pca_r0_2d_V2_res.csv';
  const imageData = await loadImagesFromCSV(csvPath, 2000);
  await loadImagesToGallery(imageData);
}

initGallery(); // Call this function at an appropriate place in your Vue component lifecycle


const animate = () => {
requestAnimationFrame(animate);
controls.update();
renderer.render(scene, camera);
};

onMounted(() => {
initThree();
animate();
});

onUnmounted(() => {
if (canvasContainer.value) {
    canvasContainer.value.removeChild(renderer.domElement);
}
});
</script>
<style scoped>
</style>