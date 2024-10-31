<template>
    <div class="h-screen w-screen">
        <div ref='threejsMap' @click="onClick" @mousemove="onDrag" @mousedown="isMouseDown = true"
            @touchstart="isMouseDown = true" @mouseup="mouseUp" @touchend="mouseUp">
            <div class="absolute top-2 h-auto w-full flex flex-row justify-between space-x-1">
                <button
                    class=" h-10 w-full bg-slate-200 rounded-lg hover:bg-slate-400 active:bg-slate-300 pointer-events-auto"
                    :class="sceneMode === 'gallery' ? 'bg-slate-400' : ''"
                    @click.stop="sceneMode = 'gallery'">gallery</button>
                <button
                    class=" h-10 w-full bg-slate-200 rounded-lg hover:bg-slate-400 active:bg-slate-300 pointer-events-auto"
                    :class="sceneMode === 'analyse' ? 'bg-slate-400' : ''"
                    @click.stop="sceneMode = 'analyse'">analyse</button>
                <button
                    class=" h-10 w-full bg-slate-200 rounded-lg hover:bg-slate-400 active:bg-slate-300 pointer-events-auto"
                    :class="sceneMode === 'generate' ? 'bg-slate-400' : ''"
                    @click.stop="sceneMode = 'generate'">generate</button>
            </div>
            <div v-if="sceneMode == 'analyse'"
                class="absolute top-1/3 right-80 h-1/2 w-fit flex flex-col justify-start items-center space-y-1 pointer-events-none">
                <button
                    class="h-10 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 active:bg-slate-300 pointer-events-auto"
                    :class="semanticLinesVisible ? 'bg-slate-400' : ''" @click.stop="showSemanticLines">semantic
                    lines</button>
                <button
                    class="h-10 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 active:bg-slate-300 pointer-events-auto "
                    :class="segmentationVisible ? 'bg-slate-400' : ''"
                    @click.stop="showSegmentationGeometry">segmentation</button>
                <button
                    class="h-10 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 active:bg-slate-300 pointer-events-auto "
                    :class="paintingSequenceVisible ? 'bg-slate-400' : ''" @click.stop="showPaintingSequence">painting
                    sequence</button>
                <button
                    class="h-10 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 active:bg-slate-300 pointer-events-auto "
                    :class="imageDepthVisible ? 'bg-slate-400' : ''" @click.stop="showImageDepth">image
                    depth</button>
                <button
                    class="h-10 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 active:bg-slate-300 pointer-events-auto "
                    :class="semanticAnalysisVisible ? 'bg-slate-400' : ''"
                    @click.stop="showSemanticAnalysis(true)">semantic
                    analysis</button>
                <button
                    class="h-10 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 active:bg-slate-300 pointer-events-auto "
                    :class="houghLinesVisible ? 'bg-slate-400' : ''" @click.stop="showSemanticAnalysis(true)">Hough
                    lines</button>
                <button
                    class="h-10 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 active:bg-slate-300 pointer-events-auto "
                    :class="houghCirclesVisible ? 'bg-slate-400' : ''" @click.stop="showSemanticAnalysis(true)">Hough
                    circles</button>
                <button
                    class="h-10 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 active:bg-slate-300 pointer-events-auto "
                    :class="shapeRecognitionVisible ? 'bg-slate-400' : ''"
                    @click.stop="showSemanticAnalysis(true)">shape
                    recognition</button>

            </div>
            <div v-if="contentSemanticAnalysis && sceneMode == 'analyse'"
                class="absolute top-1/4 right-160 h-1/2 w-108 flex flex-col justify-start items-center space-y-1 pointer-events-none text-white text-xs whitespace-pre-wrap ">
                <p>{{ contentSemanticAnalysis }}</p>

            </div>

            <div v-if="sceneMode == 'analyse'"
                class="absolute bottom-0 left-0 w-fit flex flex-row justify-start items-center space-x-1 pointer-events-none ">
                <button class="h-10 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 pointer-events-auto"
                    @click.stop="loadPreviousImage">BACK</button>
                <button class="h-10 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 pointer-events-auto"
                    @click.stop="loadNextImage">NEXT</button>
                <button class="h-10 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 pointer-events-auto"
                    @click.stop="loadRandomNewImage">NEW</button>
                <button class="h-10 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 pointer-events-auto"
                    @click.stop="putImageIntoUserGallery">LIKE</button>
            </div>

            <div v-if="sceneMode == 'gallery'"
                class="absolute top-1/3 left-16 w-fit flex flex-col justify-start items-start space-y-1 pointer-events-none ">
                <button class="h-10 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 pointer-events-auto"
                    :class="gallery == 'demo' ? 'bg-slate-400' : ''"
                    @click.stop="() => { gallery = 'demo'; loadGallery(); }">demogallery</button>
                <button v-show="userGalleryGroup.children.length > 0"
                    class="h-10 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 pointer-events-auto"
                    :class="gallery == 'user' ? 'bg-slate-400' : ''"
                    @click.stop="() => { gallery = 'user'; loadGallery(); }">usergallery</button>
                <!-- Button to load custom gallery -->
                <button class="h-10 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 pointer-events-auto"
                    @click.stop="triggerFileInput">load custom gallery</button>
                <input type="file" ref="fileInput" @change="handleFileChange" style="display: none;" accept=".json">

                <button class="h-10 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 pointer-events-auto"
                    @click.stop="downloadGallery">save gallery</button>
                <button class="h-10 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 pointer-events-auto"
                    @click.stop="requestOllamaApi">tell me a joke</button>
            </div>

        </div>

    </div>
</template>
<script setup lang="ts">
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import * as TWEEN from '@tweenjs/tween.js';
import { onMounted, ref, watch } from 'vue';
import { Line2 } from 'three/examples/jsm/lines/Line2.js';
import { LineGeometry } from 'three/examples/jsm/lines/LineGeometry.js';
import { LineMaterial } from 'three/examples/jsm/lines/LineMaterial.js';
import Stats from 'three/examples/jsm/libs/stats.module.js';
//import { depth } from 'three/examples/jsm/nodes/Nodes.js';
//import { randFloat } from 'three/src/math/MathUtils.js';
import { randInt } from 'three/src/math/MathUtils.js';
//import { cos } from 'three/examples/jsm/nodes/Nodes.js';
//import { CSS2DObject } from 'three/examples/jsm/renderers/CSS2DRenderer.js';
//import { CSS2DRenderer } from 'three/examples/jsm/renderers/CSS2DRenderer.js';



const fileInput = ref<HTMLInputElement | null>(null);

// Create a scene
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x707070);

// Create a camera
const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
camera.position.z = 5;

// Create a renderer
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.outputColorSpace = THREE.LinearSRGBColorSpace;
renderer.setSize(window.innerWidth, window.innerHeight);
//renderer.toneMapping = THREE.ACESFilmicToneMapping
renderer.toneMapping = 0

// Create controls
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;

// Create raycaster
const raycaster = new THREE.Raycaster();

const threejsMap = ref<Node>()

const domElement = renderer.domElement;

const semanticLinesVisible = ref(false)
const segmentationVisible = ref(false)
const paintingSequenceVisible = ref(false)
const imageDepthVisible = ref(false)
const semanticAnalysisVisible = ref(false)
const houghLinesVisible = ref(false)
const houghCirclesVisible = ref(false)
const shapeRecognitionVisible = ref(false)
const contentSemanticAnalysis = ref(''); // Declare a ref variable with an initial empty string

// this has to be deleted ASAP
// <button class="h-10 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 pointer-events-auto"
//                     @click.stop="() => { emptyGroup(userGalleryGroup); gallery = 'user'; loadGallery(); }">load custom
//                     gallery</button>


//STATS for FPS monitoring
const stats = new Stats();
stats.showPanel(0); // 0: fps, 1: ms, 2: mb, 3+: custom
document.body.appendChild(stats.dom);

onMounted(() => {
    stats.begin();
    threejsMap.value?.appendChild(domElement);
    //loadGallery();
    loadAnalyse(); //is this correct here? - otherwise we do not get an image on first startup, only when we change the state of scenemode
    window.addEventListener("resize", setSize);
    setSize();
    animate();
})

const setSize = () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();

    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setPixelRatio(window.devicePixelRatio);
};


// Render the scene
function animate() {
    stats.update();
    requestAnimationFrame(animate);
    renderer.render(scene, camera);
    controls.update();
    TWEEN.update();
}
//const sceneMode = ref<'gallery' | 'analyse' | 'generate'>('gallery')
const sceneMode = ref<'gallery' | 'analyse' | 'generate'>('analyse')
//make a variable which discerns whether we are working with the demo gallery or the user gallery
const gallery = ref<'demo' | 'user'>('demo')


watch(() => sceneMode.value, (newVal, oldVal) => {
    if (newVal === oldVal) return
    if (newVal === 'gallery') {
        loadGallery()
    } else if (newVal === 'analyse') {
        loadAnalyse()
    } else if (newVal === 'generate') {
        loadGenerate()
    }
})

//we will make 2 gallery groups - one for the demo gallery and one for the user gallery
const demoGalleryGroup: THREE.Group = new THREE.Group();
demoGalleryGroup.name = 'demoGalleryGroup'
scene.add(demoGalleryGroup);
const userGalleryGroup: THREE.Group = new THREE.Group();
userGalleryGroup.name = 'userGalleryGroup'
scene.add(userGalleryGroup);
const analyseGroup: THREE.Group = new THREE.Group();
analyseGroup.name = 'analyseGroup'
scene.add(analyseGroup);
const generateGroup: THREE.Group = new THREE.Group();
generateGroup.name = 'generateGroup'
scene.add(generateGroup);


const hideGroup = (groups: THREE.Group[]) => {
    groups.forEach(group => {
        group.children.forEach(child => {
            child.visible = false
        })
    })
}
const showGroup = (groups: THREE.Group[]) => {
    groups.forEach(group => {
        group.children.forEach(child => {
            child.visible = true
        })
    })
}

//even more updated third attempt of emptying a group
const disposeObjects = (object: THREE.Object3D) => {
    object.traverse((child) => {
        if (child instanceof THREE.Mesh) {
            console.log('dispose:', child);
            child.geometry.dispose();
            child.material.dispose();
            if (child.material.map) child.material.map.dispose(); // Dispose of the texture if it exists
        }
    });
}

const emptyGroup = (targetGroup: THREE.Group) => {
    while (targetGroup.children.length > 0) {
        const object = targetGroup.children[0];
        targetGroup.remove(object);
        disposeObjects(object); // Use disposeObjects to handle disposal
    }
};


type ImageData = {
    file_name: string;
    resolution: {
        width: number;
        height: number;
    };
};


//new version below
const loadGalleryImages = async (galleryFile: string, targetGroup: THREE.Group) => {
    // Clear the target group
    // while (targetGroup.children.length > 0) {
    //     const object = targetGroup.children[0];
    //     targetGroup.remove(object);
    //     if (object instanceof THREE.Mesh) {
    //         object.geometry.dispose();
    //     }
    //     if (object instanceof THREE.Mesh) {
    //         object.material.dispose();
    //     }
    //     if (object instanceof THREE.Mesh && object.material.map) object.material.map.dispose(); // Dispose of the texture if it exists
    // }
    emptyGroup(targetGroup);


    // Fetch and load the gallery images
    const response = await fetch(galleryFile);
    const data = await response.json();

    data.forEach((image: ImageData) => {
        const geometry = new THREE.PlaneGeometry(1, 1);
        const texture = new THREE.TextureLoader().load(`/sample_images_10k_orig01/${image.file_name}`);
        const material = new THREE.MeshBasicMaterial({ map: texture, side: THREE.DoubleSide });
        const plane = new THREE.Mesh(geometry, material);
        plane.scale.set(image.resolution.width / image.resolution.height, 1, 1);
        plane.visible = true;
        plane.userData = { image };
        plane.userData.isImage = true;
        plane.name = image.file_name;
        targetGroup.add(plane);
    });

    arrangeIn2DGrid(10, 0.1, targetGroup.children);
};


//old version below
// const loadGalleryImages = async (galleryFile: string) => {
//     await fetch(galleryFile)
//         .then(response => response.json())
//         .then(data => {
//             data.forEach((image: ImageData) => {
//                 const geometry = new THREE.PlaneGeometry(1, 1);
//                 const texture = new THREE.TextureLoader().load(`/sample_images_10k_orig01/${image.file_name}`);
//                 const material = new THREE.MeshBasicMaterial({ map: texture, side: THREE.DoubleSide });
//                 //const material = new THREE.MeshLambertMaterial({ map: texture, side: THREE.DoubleSide });
//                 const plane = new THREE.Mesh(geometry, material);
//                 plane.scale.set(image.resolution.width / image.resolution.height, 1, 1);
//                 plane.visible = true
//                 //plane.translateX(offset.value) //NOT REQUIRED??
//                 plane.userData = { image }
//                 plane.userData.isImage = true;
//                 plane.name = image.file_name;

//                 if (gallery.value === 'user') {
//                     userGalleryGroup.add(plane);
//                 }
//                 else {
//                     demoGalleryGroup.add(plane);
//                 }
//             });
//         });
//     if (gallery.value === 'demo') {
//         arrangeIn2DGrid(10, 0.1, demoGalleryGroup.children)
//     }
//     else if (gallery.value === 'user') {
//         arrangeIn2DGrid(10, 0.1, userGalleryGroup.children)

//     }
//     //arrangeIn2DGrid(10, 0.1, demoGalleryGroup.children) //why are we doing this only for the demo gallery? - we should do it for the user gallery as well
// }

const isDemoGalleryLoaded = ref(false)
const loadGallery = async () => {

    if (gallery.value === 'demo') {
        hideGroup([userGalleryGroup, analyseGroup, generateGroup]);
        if (isDemoGalleryLoaded.value) {
            showGroup([demoGalleryGroup]);
            tweenToPosition(lastCameraPosition);
            tweenToCameraTarget(controls.target, lastCameraTarget);
            return;
        }
        else {
            await loadGalleryImages('/image_resolutions100.json', demoGalleryGroup);
            //loadGalleryImages('/image_resolutions100.json');
            //arrangeIn2DGrid(10, 0.1, demoGalleryGroup.children);
            isDemoGalleryLoaded.value = true;
        }

    } else if (gallery.value === 'user') {
        if (userGalleryGroup.children.length > 0) {
            showGroup([userGalleryGroup]);
            arrangeIn2DGrid(10, 0.1, userGalleryGroup.children)
            tweenToPosition(lastCameraPosition);
            tweenToCameraTarget(controls.target, lastCameraTarget);
            hideGroup([demoGalleryGroup, analyseGroup, generateGroup]);
            return;
        }
        await loadGalleryImages('/user_gallery02.json', userGalleryGroup);
        //loadGalleryImages('/user_gallery02.json');
        //isDemoGalleryLoaded.value = true;
    }
    sceneMode.value = 'gallery'
    hideGroup([analyseGroup, generateGroup]) //not sure if we need this
}

const arrangeIn2DGrid = (maxWidth: number, margin: number, group: THREE.Object3D[]) => {
    let i = 0;
    // const cols = Math.ceil(group.length / rows)
    let accumulatedHeight = 0;
    for (let row = 0; row < 100; row++) {
        let accumulatedWidth = 0;
        for (let col = 0; col < 100; col++) {
            if (i >= group.length) {
                break
            }
            const obj = group[i];
            const width = obj.scale.x;
            accumulatedWidth += width / 2;
            obj.position.set(accumulatedWidth, accumulatedHeight, 0);
            accumulatedWidth += width / 2 + margin;
            i++;
            if (accumulatedWidth > maxWidth) {
                break
            }
        }
        accumulatedHeight -= 1 + margin;
    }
}


//new version with await before loadGalleryImages
// const loadCustomGallery = async () => {
//     console.log('load custom gallery');
//     //emptyGroup(userGalleryGroup); // lets try without this here - since we are calling it anyway in the loadGalleryImgages function
//     gallery.value = 'user';
//     await loadGalleryImages('/user_gallery01.json', userGalleryGroup);
//     //loadGalleryImages('/user_gallery01.json');
//     //loadGallery(); //not required
//     sceneMode.value = 'gallery'
//     hideGroup([analyseGroup, generateGroup, demoGalleryGroup]);
// };

//even newer version with user specified gallery file
const loadCustomGallery = async (filePath: string) => {
    console.log('load custom gallery');
    gallery.value = 'user';
    await loadGalleryImages(filePath, userGalleryGroup);
    sceneMode.value = 'gallery';
    hideGroup([analyseGroup, generateGroup, demoGalleryGroup]);
};

const triggerFileInput = () => {
    fileInput.value?.click();
};

const handleFileChange = (event: Event) => {
    const input = event.target as HTMLInputElement;
    const file = input.files?.[0];
    if (file) {
        const filePath = URL.createObjectURL(file);
        loadCustomGallery(filePath);
    }
};

const downloadGallery = () => {
    // Step 1: Extract Data from userGalleryGroup
    const userData = userGalleryGroup.children.map(object => ({
        file_name: object.userData.image.file_name,
        resolution: {
            width: object.userData.image.resolution.width,
            height: object.userData.image.resolution.height
        }
    }));

    // Step 2: Convert data to JSON
    const jsonData = JSON.stringify(userData, null, 2);
    const blob = new Blob([jsonData], { type: "application/json" });
    const url = URL.createObjectURL(blob);

    // Step 3: Trigger Download
    const downloadLink = document.createElement("a");
    (downloadLink as HTMLAnchorElement).href = url;
    downloadLink.download = "userGalleryData.json";
    document.body.appendChild(downloadLink);
    if (confirm("Do you want to download the gallery data?")) {
        downloadLink.click();
    }
    document.body.removeChild(downloadLink);
    URL.revokeObjectURL(url);
    console.log('downloaded:', userData);
};


const requestOllamaApi = async () => {
    fetch('https://seahorse-polished-remarkably.ngrok-free.app/api/generate', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
        },
        body: JSON.stringify({

            "model": "llama3.1:70b",
            "temperature": 0,
            "prompt": "Tell me a joke!",
            "system": "You are an ironic Programmer.",
            "stream": false
        }), // Replace with actual data to send
    })
        .then(response => {
            if (!response.ok) {
                throw new Error('Network response was not ok ' + response.statusText);
            }
            return response.json();
        })
        .then(data => {
            console.log('Data saved:', data);
            // Optionally do something with the response data
        })
        .catch(error => {
            console.error('There was a problem with the fetch operation:', error);
        });
}


//create a function that loads a random image from a folder and displays it in the analyse mode
//this function should be called when the analyse mode is activated
let loadedImages: number[] = []; // Store filenames or indices of loaded images
let imagesData: ImageData[] = [];   // Store the parsed JSON data here
let currentImageIndex: number = -1;  // Tracks the index of the current image (-1 means no image loaded)

// Function to load and parse the JSON file (metadata)
async function loadImagesMetadata() {
    const response = await fetch('/image_resolutions10k.json');
    const data = await response.json();
    imagesData = data; // Save the parsed data
    return data;
}

// Helper function to get a random index that hasn't been used
function getRandomUnusedIndex() {
    if (loadedImages.length === imagesData.length) {
        console.log('All images have been loaded at least once.');
        return null; // All images have been used
    }

    let randomIndex;
    do {
        randomIndex = Math.floor(Math.random() * imagesData.length);
    } while (loadedImages.includes(randomIndex));

    return randomIndex;
}

// Function to load an image at a specific index
function loadImageByIndex(index: number) {
    //here we  should hide the whole analysegroup again, to hide the previous image and its analysis results
    //then we will load a new image and show the analysis results for this image - at the end after loading - and updating the activeimage value
    hideGroup([analyseGroup, userGalleryGroup]); //the hiding of the userGalleryGroup is necessary, when liking the image, otherwise it will be visible when next image is loaded, since it is added to userGalleryGroup
    const image = imagesData[loadedImages[index]];
    console.log('index:', index);
    console.log('loadedImages[index]:', loadedImages[index]);
    console.log('image.file_name:', image.file_name);

    //first we will check, if the image has already been loaded and is thus part of the analyseGroup
    // this will be always the case, when we switch between images unless we later remove images from analyseGroup due to memory issues
    //only the random image loading will not be part of the analyseGroup
    let found = false;
    analyseGroup.children.forEach(child => {
        if (child.name === image.file_name) {
            child.visible = true;
            found = true;
            activeImage.value = child;
            //console.log('we found the image in the analyseGroup:', child);
            //console.log('userGalleryGroup:', userGalleryGroup);
        }
    })
    //if the image is not found in the analyseGroup, we will load it and add it to the analyseGroup
    if (!found) {
        console.log('we did not find the image in the analyseGroup - loading it now');
        const geometry = new THREE.PlaneGeometry(1, 1);
        const texture = new THREE.TextureLoader().load(`/sample_images_10k_orig01/${image.file_name}`);
        const material = new THREE.MeshBasicMaterial({ map: texture, side: THREE.DoubleSide });
        const plane = new THREE.Mesh(geometry, material);
        plane.scale.set(image.resolution.width / image.resolution.height, 1, 1);
        plane.visible = true
        plane.userData = { image }
        plane.userData.isImage = true;
        plane.name = image.file_name;
        analyseGroup.add(plane);
        //console.log('plane:', plane);
        activeImage.value = plane // Set the active image to the newly loaded image
    }
    // Update the current image index
    currentImageIndex = index;
    //move the camera position - here we should improve by checking if the semantic analysis is visible and if yes move directly to the cameraSlide position
    if (activeImage.value) {
        moveCameraToPosition(activeImage.value.position);
    }
    //show the analysis results for the new image
    //we should make it conditional, so that we only show the analysis results, if the corresponding checkbox is checked
    if (semanticLinesVisible.value) {
        showSemanticLines();
        //console.log('semanticLinesVisible: ', semanticLinesVisible.value);
    }
    if (segmentationVisible.value) {
        showSegmentationGeometry();
    }
    if (paintingSequenceVisible.value) {
        showPaintingSequence();
    }
    if (imageDepthVisible.value) {
        showImageDepth();
    }
    if (semanticAnalysisVisible.value) {
        showSemanticAnalysis(false);
        //console.log('semanticAnalysisVisible: ', semanticAnalysisVisible.value);
    }
    if (houghLinesVisible.value) {
        //showHoughLines();
        showSemanticAnalysis(false);
    }
    if (houghCirclesVisible.value) {
        //showHoughCircles();
        showSemanticAnalysis(false);
    }
    if (shapeRecognitionVisible.value) {
        //showShapeRecognition();
        showSemanticAnalysis(false);
    }

    console.log('activeImage:', activeImage.value);
}

const loadRandomNewImage = async () => {
    if (imagesData.length === 0) {
        await loadImagesMetadata(); // Ensure metadata is loaded
    }

    const randomIndex = getRandomUnusedIndex();
    if (randomIndex === null) {
        return; // No more images to load
    }

    // Add the index of the loaded image to the array of loaded images
    loadedImages.push(randomIndex);

    // Load the new image by index
    loadImageByIndex(loadedImages.length - 1);
    //console.log('loadedImages:', loadedImages);
}

// Function for "Back" button functionality
function loadPreviousImage() {
    if (currentImageIndex > 0) {
        loadImageByIndex(currentImageIndex - 1);
    } else {
        console.log("No previous image available.");
    }
}

// Function for "Next" button functionality
function loadNextImage() {
    if (currentImageIndex < loadedImages.length - 1) {
        loadImageByIndex(currentImageIndex + 1);
    } else {
        console.log("Already the latest image, click NEW for new image.");
    }
}

// function to put the current image into the user gallery
function putImageIntoUserGallery() {
    if (currentImageIndex === -1) {
        console.log("No image loaded to put into user gallery.");
        return;
    }

    // // Get the current image which is the active image

    if (activeImage.value) {
        //check whether the image is already in the user gallery
        let found = false;
        userGalleryGroup.children.forEach(child => {
            if (child.name === activeImage.value?.userData.image.file_name) {
                found = true;
            }
        })
        if (found) {
            console.log("Image is already in the user gallery.");
            return;
        }
        else {
            userGalleryGroup.add(activeImage.value.clone());
        }
        gallery.value = 'user';
    } else {
        console.log("No active image to add to the user gallery.");
    }

}

const loadAnalyse = () => {
    lastCameraPosition.copy(camera.position) //will this work also - if we directly start in analyse mode? i think yes, since we declare it as a global variable
    lastCameraTarget.copy(controls.target)
    hideGroup([demoGalleryGroup, userGalleryGroup, generateGroup])
    semanticLinesVisible.value = false;
    segmentationVisible.value = false;
    paintingSequenceVisible.value = false;
    imageDepthVisible.value = false;
    semanticAnalysisVisible.value = false;
    contentSemanticAnalysis.value = ""; // Update the ref's value
    stopTyping(); // Stop the typewriter effect //am not sure - if this is good here - there seems to be a lot of redundancy, especially around that typewriter effect
    houghLinesVisible.value = false;
    houghCirclesVisible.value = false;
    shapeRecognitionVisible.value = false;

    //do we need this next loop?	- we already call the function hideGroup above - ok but we do not call it for the analysegroup - so why does the analyse stuff get hidden?
    // i think it is because we call the function hidegroup for the analysegroup when we go into any other scene mode - like gallery or generate
    // analyseGroup.children.forEach(child => {
    //     //analyseGroup.remove(child)
    //     // dispose everything from child 
    //     child.visible = false;
    // })

    if (activeImage.value) {
        console.log('am in analyse mode: ', activeImage.value); //to be deleted later
        let found = false;
        //console.log('userGallerGroup.length', userGalleryGroup.children.length);
        analyseGroup.children.forEach(child => {
            if (child.name === activeImage.value?.userData.image.file_name) {
                child.visible = true;
                found = true;
            }

        })
        if (!found) {
            const image = activeImage.value.clone();
            image.visible = true;
            analyseGroup.add(image);
            //console.log('image not found in analyeGroup hence added to analyseGroup: ', image);
        }

        //for each analyseGroup#

        //analyseGroup.children[0].visible = true
        moveCameraToPosition(activeImage.value.position)
    }

    else {
        console.log('no active image value - loading new random image'); //to be deleted later
        loadRandomNewImage();
    }
    // this part does not work yet - we have to create a random image loading routine from the whole of the dataset, not only from the gallery
    // probably this part should be removed later in it current form
    // else {
    //     activeImage.value = galleryGroup.children[randFloat(0, galleryGroup.children.length - 1)]
    //     console.log('am in analyse mode: ',activeImage.value);
    // }
}

const lastCameraPosition = new THREE.Vector3(0, 0, 5) //set initial camera position has to be reconsidered - but still better than leaving it empty, otherwise zooms into nothing
const lastCameraTarget = new THREE.Vector3(0, 0, 0)
const moveCameraToPosition = (position: THREE.Vector3) => {
    //this needs to be moved out of this function - but then called every time - we want to remember the last camera position
    // for now, i already moved it on top of the loadAnalyse function
    // lastCameraPosition.copy(camera.position)
    // lastCameraTarget.copy(controls.target)
    //same goes for the target position - moved it out of this function to the top of the loadAnalyse function
    const targetPosition = position.clone()
    const targetLookat = position.clone()
    const currentLookat = controls.target.clone()
    targetPosition.z += 2
    tweenToPosition(targetPosition)
    tweenToCameraTarget(currentLookat, targetLookat)
}

const tweenToPosition = (position: THREE.Vector3) => {
    const objectPosition = camera.position.clone()
    new TWEEN.Tween(objectPosition)
        .to(position, 1000)
        .easing(TWEEN.Easing.Sinusoidal.InOut)
        .onUpdate(() => {
            camera.position.copy(objectPosition)
        })
        .start()
}
const tweenToCameraTarget = (startingLookat: THREE.Vector3, targetLookat: THREE.Vector3) => {
    new TWEEN.Tween(startingLookat)
        .to(targetLookat, 1000)
        .easing(TWEEN.Easing.Sinusoidal.InOut)
        .onUpdate(() => {
            controls.target.copy(startingLookat)
            controls.update()
        })
        .start()
}
const loadGenerate = () => {
    sceneMode.value = 'generate'
    const geometry = new THREE.BoxGeometry(1, 1, 1);
    const material = new THREE.MeshBasicMaterial({ color: 0x0000ff });
    //const material = new THREE.MeshBasicMaterial();
    const cube = new THREE.Mesh(geometry, material);

    generateGroup.add(cube);
    hideGroup([demoGalleryGroup, userGalleryGroup, analyseGroup])
}


const onDrag = (event: MouseEvent) => {
    if (isMouseDown.value) {
        const pixelDistance = Math.abs(event.movementX) + Math.abs(event.movementY);
        if (pixelDistance > 5) {
            isDragging.value = true
        }
    }

}
const mouseUp = () => {
    isMouseDown.value = false
    setTimeout(() => {
        isDragging.value = false
    }, 50);
}

const isMouseDown = ref(false)
const isDragging = ref(false)
const activeImage = ref<THREE.Object3D | null>(null)
const onClick = (e: Event) => {
    const mouse = new THREE.Vector2();
    mouse.x = (e as MouseEvent).clientX / window.innerWidth * 2 - 1;
    mouse.y = -(e as MouseEvent).clientY / window.innerHeight * 2 + 1;
    raycaster.setFromCamera(mouse, camera);

    //first filter visible objects before raycasting, otherwise it gave some erratic behaviour
    const visibleObjects = [...demoGalleryGroup.children, ...userGalleryGroup.children].filter(obj => obj.visible);
    const intersects = raycaster.intersectObjects(visibleObjects);

    //const intersects = raycaster.intersectObjects([...demoGalleryGroup.children, ...userGalleryGroup.children]);
    if (intersects.length > 0 && !isDragging.value) {
        const object = intersects[0].object;
        //if (object.userData.isImage && object.visible) {  //maybe  we can make do without the visible check? YES delete this asap
        //in future we should try to toggle the raycasting on and off for the different groups - so that we can avoid the visible check
        //should use raycast = null for the groups we do not want to raycast
        if (object.userData.isImage) {
            activeImage.value = object
            console.log('object: ', object);
            if (gallery.value === 'user') {
                emptyGroup(analyseGroup);
            }
            sceneMode.value = 'analyse'
        }
    }
}

const addLine2 = (points: number[]) => {
    const geometry = new LineGeometry();
    geometry.setPositions(points);

    const material = new LineMaterial({
        //color: 0xffffff,
        color: 0xff69b4,
        linewidth: 5, // in pixels
    });

    const line = new Line2(geometry, material);
    line.computeLineDistances();
    line.scale.set(1, 1, 1);

    return line;
};

const showSemanticLines = async () => {
    let semanticLineGroup = scene.getObjectByName('semanticLineGroup_' + activeImage.value?.userData.image.file_name);
    console.log('semanticLineGroup: ', semanticLineGroup);
    console.log('analyseGroup: ', analyseGroup);
    if (semanticLineGroup) {
        console.log('semantic lines found');
        console.log('semanticLineGroup.visible: ', semanticLineGroup.visible);
        semanticLineGroup.visible = !semanticLineGroup.visible
        semanticLinesVisible.value = semanticLineGroup.visible
        console.log('semanticLineGroup.visible: ', semanticLineGroup.visible);
        return;
    }
    else {
        semanticLineGroup = new THREE.Group();
        semanticLineGroup.name = 'semanticLineGroup_' + activeImage.value?.userData.image.file_name;
        analyseGroup.add(semanticLineGroup);
        semanticLinesVisible.value = true
    }
    const image = activeImage.value?.userData.image;
    const imagePosition = activeImage.value?.position; // Get the position of the image plane
    // Get image dimensions
    const imageHeight = image.resolution.height;

    // fetch semantic lines from json file 
    const folderName = 'deep_hough_10k_orig01_out01_jsons'
    const fileName = activeImage.value?.userData.image.file_name.split('.')[0]
    const points: number[][] = [];
    const aspectRatio = activeImage.value?.scale.x
    if (aspectRatio === undefined) {
        return
    }
    await fetch(`/${folderName}/${fileName}.json`)
        .then(response => response.json())
        .then(data => {
            data.forEach((line: any) => {

                points.push([
                    line.y1 / imageHeight + (imagePosition?.x ?? 0) - 0.5 * aspectRatio, (imageHeight - line.x1) / imageHeight - 0.5 + (imagePosition?.y ?? 0), 0.01,
                    line.y2 / imageHeight + (imagePosition?.x ?? 0) - 0.5 * aspectRatio, (imageHeight - line.x2) / imageHeight - 0.5 + (imagePosition?.y ?? 0), 0.01
                ])
            });

        });

    points.forEach((line: number[]) => {
        const lineObject = addLine2(line);
        semanticLineGroup.add(lineObject);
    });
}

type ContourObject = {
    contour_number: number,
    id: number,
    area: number,
    coords: number[][],
    color: number[]
}

type CountourLayer = {
    layer: string,
    layer_number: number,
    contours: ContourObject[]
}

const showSegmentationGeometry = async () => {
    if (!activeImage.value) {
        return
    }
    let segmentationGroup = scene.getObjectByName('segmentationGroup_' + activeImage.value?.userData.image.file_name);
    if (segmentationGroup) {
        segmentationGroup.visible = !segmentationGroup.visible
        segmentationVisible.value = segmentationGroup.visible
        return;
    }
    else {
        segmentationGroup = new THREE.Group();
        segmentationGroup.name = 'segmentationGroup_' + activeImage.value?.userData.image.file_name;
        analyseGroup.add(segmentationGroup);
        segmentationVisible.value = true;
    }
    const image = activeImage.value?.userData.image;
    const imagePosition = activeImage.value?.position; // Get the position of the image plane

    const folderName = 'sam_out01_contours_10k_orig01_cleanup_merged_100/'
    const fileName = activeImage.value?.userData.image.file_name.replace('.', '_') + '.json'

    const contourJson: CountourLayer[] = await fetch(`/${folderName}/${fileName}`)
        .then(response => response.json())
        .then(data => {
            return data
        });

    contourJson.forEach((contourLayer: CountourLayer, index) => {
        const offset = 0.5 / contourJson.length

        const segmentationLayerGroup = createSegmentationShape(contourLayer, imagePosition, image)
        segmentationLayerGroup.position.z += 0.01 + index * offset;
        segmentationGroup.add(segmentationLayerGroup);

    });

}


const createSegmentationShape = (contourLayer: CountourLayer, imagePosition: THREE.Vector3, image: ImageData, color?: THREE.Color) => {
    const layerContourGroup = new THREE.Group();
    contourLayer.contours.forEach((contour) => {

        const shape = new THREE.Shape();
        shape.arcLengthDivisions = 1;

        contour.coords.forEach((point, index) => {
            // Normalize coordinates
            const x = point[0] / image.resolution.width;
            const y = (image.resolution.height - point[1]) / image.resolution.height;

            if (index === 0) {
                shape.moveTo(x, y);
            } else {
                shape.lineTo(x, y);
            }
        });
        //draw contour lines from shape - optional to be done later
        let material = new THREE.MeshBasicMaterial({ color: 0x00ff00, side: THREE.DoubleSide });
        if (color) {
            material = new THREE.MeshBasicMaterial({ color: color, side: THREE.DoubleSide });
        }
        else {
            activeImage.value?.traverse((child) => {
                if (child instanceof THREE.Mesh) {
                    material = child.material
                }
            });
        }

        const geometry = new THREE.ShapeGeometry(shape);
        const mesh = new THREE.Mesh(geometry, material);
        mesh.scale.set(image.resolution.width / image.resolution.height, 1, 1);
        const meshPosition = new THREE.Vector3();
        meshPosition.copy(imagePosition as THREE.Vector3);
        const offSetVector = new THREE.Vector3(-0.5 * image.resolution.width / image.resolution.height, -0.5, 0);
        meshPosition.add(offSetVector);
        mesh.position.copy(meshPosition as THREE.Vector3);
        mesh.visible = true;
        layerContourGroup.add(mesh);
        debugger

    });

    return layerContourGroup;
}

const showPaintingSequence = async () => {
    if (!activeImage.value) {
        return
    }
    let paintingSequenceGroup = scene.getObjectByName('paintingSequenceGroup_' + activeImage.value?.userData.image.file_name);
    if (paintingSequenceGroup) {
        paintingSequenceGroup.visible = !paintingSequenceGroup.visible
        paintingSequenceVisible.value = paintingSequenceGroup.visible
        return;
    }
    else {
        paintingSequenceGroup = new THREE.Group();
        paintingSequenceGroup.name = 'paintingSequenceGroup_' + activeImage.value?.userData.image.file_name;
        analyseGroup.add(paintingSequenceGroup);
        paintingSequenceVisible.value = true;
    }
    const image = activeImage.value?.userData.image;
    const imagePosition = activeImage.value?.position; // Get the position of the image plane

    const folderName = 'learn2paint_out01_contours_cleanup_merged_100/'
    const fileName = activeImage.value?.userData.image.file_name.split('.')[0] + '.json'


    const contourJson: CountourLayer[] = await fetch(`/${folderName}/${fileName}`)
        .then(response => response.json())
        .then(data => {
            return data
        });

    contourJson.forEach((contourLayer: CountourLayer, index) => {
        const offset = 0.5 / contourJson.length
        const r = contourLayer.contours[0].color[0]; //this is only possible if there is only one contour per layer
        const g = contourLayer.contours[0].color[1]; //this is only possible if there is only one contour per layer
        const b = contourLayer.contours[0].color[2]; //this is only possible if there is only one contour per layer
        const color = new THREE.Color(`rgb(${r}, ${g}, ${b})`);
        const paintingSequenceLayerGroup = createSegmentationShape(contourLayer, imagePosition, image, color)
        paintingSequenceLayerGroup.position.z += 0.008 + index * offset; //here the arbitrary value of 0.008 is added to avoid overlapping with the segmentation geometry
        paintingSequenceGroup.add(paintingSequenceLayerGroup);

    });

}

const showImageDepth = async () => {
    if (!activeImage.value) {
        return
    }
    let imageDepthGroup = scene.getObjectByName('imageDepthGroup_' + activeImage.value?.userData.image.file_name);
    if (imageDepthGroup) {
        imageDepthGroup.visible = !imageDepthGroup.visible
        imageDepthVisible.value = imageDepthGroup.visible
        return;
    }
    else {
        imageDepthGroup = new THREE.Group();
        imageDepthGroup.name = 'imageDepthGroup_' + activeImage.value?.userData.image.file_name;
        analyseGroup.add(imageDepthGroup);
        imageDepthVisible.value = true;
    }
    const image = activeImage.value?.userData.image;
    const imagePosition = activeImage.value?.position; // Get the position of the image plane

    const folderName = 'midas_3_hybrid_magma/'

    // fetch depth map from image file 
    const depthFileName = activeImage.value?.userData.image.file_name.split('.')[0] + '.png'
    const depthGeometry = new THREE.PlaneGeometry(1, 1);
    const depthTexture = new THREE.TextureLoader().load(`/${folderName}/${depthFileName}`);
    const depthMaterial = new THREE.MeshBasicMaterial({ map: depthTexture, side: THREE.DoubleSide, transparent: true, opacity: 0.7 });
    const depthPlane = new THREE.Mesh(depthGeometry, depthMaterial);
    depthPlane.scale.set(image.resolution.width / image.resolution.height, 1, 1);
    depthPlane.position.copy(imagePosition as THREE.Vector3);
    depthPlane.position.z += 0.005;
    depthPlane.visible = true;
    depthPlane.userData = { image }
    depthPlane.userData.isImage = true;
    depthPlane.name = image.file_name;
    imageDepthGroup.add(depthPlane);

    //below is the next step to create a point cloud from the depth image
    // const depthPoints: number[] = [];
    // depthPoints.forEach((depth, index) => {
    //     const x = index / image.resolution.width;
    //     const y = (image.resolution.height - depth) / image.resolution.height;
    //     depthPoints.push(x, y, 0.01);
    // });

}

// Typewriter effect setup
let typingTimeout: ReturnType<typeof setTimeout>; // Variable to store the timeout ID
let isTyping = false; // Flag to track if typing is in progress

// Function to apply Markdown formatting
function applyMarkdownFormatting(word: string) {
    return word.replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>');
}

// Function to type the next word
function typeNextWord(words: string[], wordIndex: number) {
    if (wordIndex < words.length && isTyping) {
        //if (wordIndex < words.length) {
        let currentWord = words[wordIndex];
        const formattedWord = applyMarkdownFormatting(currentWord);
        contentSemanticAnalysis.value += formattedWord;
        typingTimeout = setTimeout(() => typeNextWord(words, wordIndex + 1), 10); // Adjust typing speed
    }
}

// Function to start the typewriter effect
function startTyping(newText: string) {
    stopTyping(); // Stop the current typing process if it's running
    const words = newText.split(/(\s+|\n+)/); // Split the new text by spaces and newlines
    contentSemanticAnalysis.value = ""; // Clear existing text
    isTyping = true; // Set typing flag to true
    typeNextWord(words, 0); // Start typing
}

// Function to stop the typewriter effect
function stopTyping() {
    clearTimeout(typingTimeout); // Clear any existing timeout
    isTyping = false; // Set typing flag to false
}

const showSemanticAnalysis = async (toggle: boolean) => {
    if (!activeImage.value) return;

    if (semanticAnalysisVisible.value && toggle) {
        semanticAnalysisVisible.value = !semanticAnalysisVisible.value;
        contentSemanticAnalysis.value = ""; // Clear existing text
        moveCameraToPosition(activeImage.value.position);
        stopTyping(); // Stop the current typing process if it's running
        return;
    }

    if (semanticAnalysisVisible.value === true && !toggle || semanticAnalysisVisible.value === false) {
        contentSemanticAnalysis.value = ""; // Clear existing text
        stopTyping();
        const cameraSlidePosition = activeImage.value.position.clone();
        cameraSlidePosition.x += 1;
        moveCameraToPosition(cameraSlidePosition);

        const folderName = 'llava_run02_orig10k/';
        const fileName = activeImage.value?.userData.image.file_name.replace('.', '_') + '.json';
        const semanticJson = await fetch(`/${folderName}/${fileName}`).then(response => response.json());

        const randomKey = 'question' + randInt(1, 5);
        const text = semanticJson[randomKey];
        semanticAnalysisVisible.value = true; //somehow this needs to be set to true, otherwise the text will not be displayed, but it should not be like that
        startTyping(text);
    }
}




</script>
<style scoped></style>