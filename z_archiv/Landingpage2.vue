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

// Load and add images to the scene
loadImagesToGallery(['/sample_images_10k_orig01/aaron-douglas-let-my-people-go.jpg', '/sample_images_10k_orig01/aaron-douglas-untitled2.png', '/sample_images_10k_orig01/aaron-douglas-untitled2-1.png']);
};

const loadImagesToGallery = (imageUrls: string[]) => {
const loader = new THREE.TextureLoader();
imageUrls.forEach((url, index) => {
    loader.load(url, (texture) => {
    const geometry = new THREE.PlaneGeometry(1, 1);
    const material = new THREE.MeshBasicMaterial({ map: texture });
    const plane = new THREE.Mesh(geometry, material);
    plane.position.x = index * 2; // Arrange horizontally
    scene.add(plane);
    });
});
};

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