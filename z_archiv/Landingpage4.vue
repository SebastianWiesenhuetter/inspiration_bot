<template>
    <div ref="canvasContainer" class="h-screen w-screen">
        <div class="absolute inset-0  flex justify-center items-center" style="pointer-events: none;">
                <button style="pointer-events: auto;" class="h-10 px-3 w-fit bg-slate-200 rounded-lg"  @click="leave">START</button>
        </div>
    </div>
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

const loadImagesFromCSV = async (csvPath: string, n: number) => {
  const response = await fetch(csvPath);
  const csvText = await response.text();
  const lines = csvText.split('\n');

  // Identify the column indexes
  const headers = lines[0].split(',').map(header => header.trim());
  const filenameIndex = headers.indexOf('image_filename');
  const widthIndex = headers.indexOf('image_width');
  const heightIndex = headers.indexOf('image_height');
  
  if (filenameIndex === -1 || widthIndex === -1 || heightIndex === -1) {
    console.error('Required column(s) not found');
    return;
  }

  const basePath = '/sample_images_10k_orig01/'; // Fixed path to be concatenated

  // Extract image filenames, widths, and heights
  const imageData = lines.slice(1, n + 1)
    .map(line => {
      const columns = line.split(',').map(column => column.trim());
      const filename = columns[filenameIndex];
      const width = parseFloat(columns[widthIndex]);
      const height = parseFloat(columns[heightIndex]);
      return { url: basePath + filename, width, height };
    })
    .filter(data => data.url !== basePath); // Filter out any URLs that are just the base path

  for (const { url, width, height } of imageData) {
    const texture = await new Promise<THREE.Texture>((resolve, reject) => {
      new THREE.TextureLoader().load(url, resolve, undefined, reject);
    });
    const aspectRatio = width / height;
    const geometry = new THREE.PlaneGeometry(aspectRatio, 1); // Use aspect ratio to size geometry
    const material = new THREE.MeshBasicMaterial({ map: texture });
    const plane = new THREE.Mesh(geometry, material);
    scene.add(plane);
  }
};

// Example usage with CSV file, reading up to the 10th row
const csvPath = '/tsne_coordinates_pca_r0_2d_res.csv';
loadImagesFromCSV(csvPath, 10);



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