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
                    @click.stop="downloadGallery(userGalleryGroup, generateGalleryGroup)">save gallery</button>
                <button class="h-10 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 pointer-events-auto"
                    @click.stop="newEmptyGallery">new empty gallery</button>
                <!--<button class="h-10 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 pointer-events-auto"
                    @click.stop="requestComfyUIAPI">tell me a joke</button>-->
                <!-- @click.stop="requestOllamaApi">tell me a joke</button> -->
            </div>

            <div v-if="sceneMode == 'generate'"
                class="absolute bottom-1/3 left-2 h-1/2 w-fit flex flex-col justify-end items-center space-y-1 pointer-events-none">
                <button
                    class="h-7 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 active:bg-slate-300 pointer-events-auto text-sm"
                    :class="semanticLinesVisible ? 'bg-slate-400' : ''" @click.stop="showSemanticLines">semantic
                    lines</button>
                <button
                    class="h-7 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 active:bg-slate-300 pointer-events-auto text-sm"
                    :class="segmentationVisible ? 'bg-slate-400' : ''"
                    @click.stop="showSegmentationGeometry">segmentation</button>
                <button
                    class="h-7 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 active:bg-slate-300 pointer-events-auto text-sm"
                    :class="paintingSequenceVisible ? 'bg-slate-400' : ''" @click.stop="showPaintingSequence">painting
                    sequence</button>
                <button
                    class="h-7 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 active:bg-slate-300 pointer-events-auto text-sm"
                    :class="imageDepthVisible ? 'bg-slate-400' : ''" @click.stop="showImageDepth">image
                    depth</button>
                <button
                    class="h-7 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 active:bg-slate-300 pointer-events-auto text-sm"
                    :class="semanticAnalysisVisible ? 'bg-slate-400' : ''"
                    @click.stop="showSemanticAnalysis(true)">semantic
                    analysis</button>
                <button
                    class="h-7 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 active:bg-slate-300 pointer-events-auto text-sm "
                    :class="houghLinesVisible ? 'bg-slate-400' : ''" @click.stop="showSemanticAnalysis(true)">Hough
                    lines</button>
                <button
                    class="h-7 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 active:bg-slate-300 pointer-events-auto text-sm "
                    :class="houghCirclesVisible ? 'bg-slate-400' : ''" @click.stop="showSemanticAnalysis(true)">Hough
                    circles</button>
                <button
                    class="h-7 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 active:bg-slate-300 pointer-events-auto text-sm"
                    :class="shapeRecognitionVisible ? 'bg-slate-400' : ''"
                    @click.stop="showSemanticAnalysis(true)">shape
                    recognition</button>

            </div>

            <div v-if="sceneMode == 'generate'"
                class="absolute bottom-1/3 left-160 h-1/2 w-fit flex flex-col justify-end items-center space-y-1 pointer-events-none">
                <button
                    class="h-7 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 active:bg-slate-300 pointer-events-auto text-sm"
                    :class="semanticLinesVisible ? 'bg-slate-400' : ''"
                    @click.stop="generateSurfaceRelief('plateTectonics')">plate tectonics</button>
                <button
                    class="h-7 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 active:bg-slate-300 pointer-events-auto text-sm"
                    :class="semanticLinesVisible ? 'bg-slate-400' : ''"
                    @click.stop="generateSurfaceRelief('smoothSurface')">surface
                    relief</button>
                <button
                    class="h-7 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 active:bg-slate-300 pointer-events-auto text-sm"
                    @click.stop="requestComfyUIAPI">generate vision image</button>
            </div>

        </div>

    </div>
</template>
<script setup lang="ts">
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import * as TWEEN from '@tweenjs/tween.js';
import { onMounted, onUnmounted, ref, watch } from 'vue';
import { Line2 } from 'three/examples/jsm/lines/Line2.js';
import { LineGeometry } from 'three/examples/jsm/lines/LineGeometry.js';
import { LineMaterial } from 'three/examples/jsm/lines/LineMaterial.js';
import Stats from 'three/examples/jsm/libs/stats.module.js';
//import { depth } from 'three/examples/jsm/nodes/Nodes.js';
//import { randFloat } from 'three/src/math/MathUtils.js';
import { randInt } from 'three/src/math/MathUtils.js';
//import { clone } from 'three/examples/jsm/utils/SkeletonUtils.js';
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

// Create a light
const light = new THREE.AmbientLight(0xffffff, 1);
light.position.set(0, 0, 1);
scene.add(light);
//create dirrectional light
const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
directionalLight.position.set(20, 20, 20);
scene.add(directionalLight);



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




//STATS for FPS monitoring
const stats = new Stats();
stats.showPanel(0); // 0: fps, 1: ms, 2: mb, 3+: custom
document.body.appendChild(stats.dom);

onMounted(() => {
    stats.begin();
    threejsMap.value?.appendChild(domElement);
    //setupWebSocket(); // Initialize WebSocket connection /////////// remove in uvicorn setup
    //loadGallery();
    loadAnalyse(); //is this correct here? - otherwise we do not get an image on first startup, only when we change the state of scenemode
    window.addEventListener("resize", setSize);
    setSize();
    animate();
})

onUnmounted(() => { ///////////// remove this whole thing in uvicorn setup
    cleanupWebSocket(); // Clean up WebSocket connection on component unmount
});

const setSize = () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();

    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setPixelRatio(window.devicePixelRatio);
};


/// use this block for debugging and stepwise animation with arrow keys and space bar
// let simulationPaused = true; // Initially paused
// let stepFrame = false;       // Flag to advance by one frame


// document.addEventListener("keydown", (event) => {
//     if (event.key === " ") {
//         simulationPaused = !simulationPaused; // Toggle pause
//         console.log(`Simulation ${simulationPaused ? "paused" : "resumed"}.`);
//     } else if (event.key === "ArrowRight" && simulationPaused) {
//         stepFrame = true; // Allow one frame to advance
//         console.log("Advancing one frame.");
//     }
// });


// Render the scene
function animate() {
    stats.update();
    requestAnimationFrame(animate);
    renderer.render(scene, camera);
    controls.update();
    TWEEN.update();

    // for force graph simulation
    if (!graphStabilized && sceneMode.value === 'generateGallery') {
        updateRepulsiveForces();
        updateSpringForces();
        applyDampingAndCapVelocity();
        updatePositions();
        updateEdges();
        checkStabilization();
    }

    /// use this block for debugging and stepwise animation with arrow keys and space bar
    // if (!simulationPaused || stepFrame) {
    //     // Perform one simulation step
    //     updateRepulsiveForces();
    //     updateSpringForces();
    //     applyDampingAndCapVelocity();
    //     updatePositions();
    //     updateEdges();
    //     // updateRotations();

    //     // Reset stepFrame after advancing one frame
    //     if (stepFrame) {
    //         stepFrame = false;
    //     }
    // }

}

//const sceneMode = ref<'gallery' | 'analyse' | 'generate'>('gallery')
const sceneMode = ref<'gallery' | 'analyse' | 'generate' | 'generateGallery'>('analyse');
//make a variable which discerns whether we are working with the demo gallery or the user gallery
const gallery = ref<'demo' | 'user'>('demo');



watch(() => sceneMode.value, (newVal, oldVal) => {
    if (newVal === oldVal) return;
    if (newVal === 'gallery') {
        loadGallery();
        resetAnalysisResults();
    } else {
        // Save camera state only when leaving gallery mode
        if (oldVal === 'gallery') {
            lastCameraPosition.copy(camera.position)
            lastCameraTarget.copy(controls.target)
        }

        if (newVal === 'analyse') {
            loadAnalyse();
        } else if (newVal === 'generate') {
            loadGenerate();
        } else if (newVal === 'generateGallery') {
            loadGenerateGallery();
        }
    }
    //add an overall condition when leaving generateGallerymode to empty the linesForceGraphGroup
    if (oldVal === 'generateGallery') {
        emptyGroup(linesForceGraphGroup);
    }
}
);

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

const generateGalleryGroup: THREE.Group = new THREE.Group();
generateGalleryGroup.name = 'generateGalleryGroup';
scene.add(generateGalleryGroup);
const linesForceGraphGroup: THREE.Group = new THREE.Group();
linesForceGraphGroup.name = 'linesForceGraphGroup';
scene.add(linesForceGraphGroup);
const generateDemoGalleryGroup: THREE.Group = new THREE.Group();
generateDemoGalleryGroup.name = 'generateDemoGalleryGroup';
scene.add(generateDemoGalleryGroup);



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

//this function is used to toggle visibility more thoroughly and mainly intended to handle generated meshes
function toggleGroupVisibility(group: THREE.Group, visible: boolean): void {
    group.traverse((child) => {
        child.visible = visible;
    });
}

function toggleObjectsByName(
    group: THREE.Group,
    targetName: string,
    visible: boolean
): void {
    group.children.forEach((object) => {
        if (object.name === targetName || object.name === `${targetName}_edges`) {
            object.visible = visible;
        } else {
            object.visible = !visible;
        }
    });
}


function setGenerateGroupVisibility(
    group: THREE.Group,
    visible: boolean,
    activeImageName: string | null = null
): void {
    if (!activeImageName) {
        // Toggle entire group
        toggleGroupVisibility(group, visible);
    } else {
        // Toggle specific objects
        toggleObjectsByName(group, activeImageName, visible);
    }
}

///USAGE
// Show only objects related to the active image
// if (activeImage.value) {
//     setGenerateGroupVisibility(generateGroup, true, activeImage.value.userData.image.file_name);
// }

// // Hide the entire generateGroup
// setGenerateGroupVisibility(generateGroup, false);

//even more updated third attempt of emptying a group
const disposeObjects = (object: THREE.Object3D) => {
    object.traverse((child) => {
        if (child instanceof THREE.Mesh) {
            //console.log('dispose:', child);
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


// Recursive function to handle visibility for a child and its nested children
function setVisibilityRecursive(object: THREE.Object3D, targetName: string, isVisible: boolean) {
    if (object.userData.name === targetName) {
        object.visible = isVisible;
    } else {
        object.visible = false;
    }

    // Traverse through the object's children
    object.children.forEach(nestedChild => {
        setVisibilityRecursive(nestedChild, targetName, isVisible);
    });
}

type ImageData = {
    file_name: string;
    resolution: {
        width: number;
        height: number;
    };
};


////////
const loadGalleryImages = async (
    galleryFile: string,
    targetGroup: THREE.Group,
    meshGroup: THREE.Group
) => {
    // Empty the target groups
    emptyGroup(targetGroup);
    emptyGroup(meshGroup);

    // Fetch and parse the gallery JSON
    const response = await fetch(galleryFile);
    if (!response.ok) {
        console.error(`Failed to load gallery file: ${galleryFile}`);
        return;
    }

    const data = await response.json();


    for (const image of data) {
        // Create image plane
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

        // Deserialize and load associated meshes
        if (image.meshes) {
            for (const meshData of image.meshes) {
                const mesh = deserializeMeshWithEdges(meshData.mesh);

                // Add the mesh to the mesh group
                meshGroup.add(mesh);
                //targetGroup.add(mesh);
                //console.log('added mesh:', mesh);

                // Optionally, add the edgesMesh to the scene
                if (mesh.userData.edgesMesh) {
                    //console.log('added edgesMesh:', mesh.userData.edgesMesh);
                    meshGroup.add(mesh.userData.edgesMesh);
                    //targetGroup.add(mesh.userData.edgesMesh);
                }
            }
        }
    }

    // Arrange the images in a 2D grid
    arrangeIn2DGrid(10, 0.1, targetGroup.children);
    //Arrange the meshes behind their respective images
    // Arrange meshes behind their respective images
    //arrangeMeshesBehindImages(userGalleryGroup, generateGroup, -0.5);
    arrangeMeshesBehindImages(targetGroup, meshGroup, -1, -0.5);
    //arrangeMeshesBehindImages(targetGroup, meshGroup, 0, 0.1);
    //////////////////
    //console.log("After arrangeMeshesBehindImages:");
    // generateGalleryGroup.children.forEach((mesh) => {
    //     console.log(`Mesh: ${mesh.name}, Position:`, mesh.position);
    // });
    //showGroup([generateGroup]);
    // console.log('meshGroup.children.length:', meshGroup.children.length);
    // console.log('meshGroup:', meshGroup);



};





const isDemoGalleryLoaded = ref(false)
const loadGallery = async () => {

    if (gallery.value === 'demo') {
        hideGroup([userGalleryGroup, analyseGroup, generateGroup, generateGalleryGroup]);
        if (isDemoGalleryLoaded.value) {
            showGroup([demoGalleryGroup]);
            tweenToPosition(lastCameraPosition);
            tweenToCameraTarget(controls.target, lastCameraTarget);
            return;
        }
        else {
            //await loadGalleryImages('/image_resolutions100.json', demoGalleryGroup, generateGroup); //not sure if we need to emtpy the generateGroup or even the generateGalleryGroup for that matter
            // await loadGalleryImages('/image_resolutions100.json', demoGalleryGroup, generateGalleryGroup); // we should introduce a new group: generateDemoGalleryGroup to load the generated results of the demo gallery
            await loadGalleryImages('/image_resolutions100.json', demoGalleryGroup, generateDemoGalleryGroup);
            //await loadGalleryImages('/userGalleryData(177).json', demoGalleryGroup, generateDemoGalleryGroup); //example with geometry
            //showGroup([generateDemoGalleryGroup]);
            //console.log('generateDemoGalleryGroup:', generateDemoGalleryGroup);
            isDemoGalleryLoaded.value = true;
        }

    } else if (gallery.value === 'user') {
        if (userGalleryGroup.children.length > 0) {
            showGroup([userGalleryGroup, generateGalleryGroup]);
            //console.log('generateGalleryGroup.children.length:', generateGalleryGroup.children.length);
            arrangeIn2DGrid(10, 0.1, userGalleryGroup.children)
            arrangeMeshesBehindImages(userGalleryGroup, generateGalleryGroup, -1, -0.5);
            tweenToPosition(lastCameraPosition);
            tweenToCameraTarget(controls.target, lastCameraTarget);
            //hideGroup([demoGalleryGroup, analyseGroup, generateGroup]);
            hideGroup([demoGalleryGroup, analyseGroup, generateGroup, generateDemoGalleryGroup]);
            //showGroup([generateGroup]);
            return;
        }
        await loadGalleryImages('/user_gallery01.json', userGalleryGroup, generateGroup); //this case should never really occur right?
        //loadGalleryImages('/user_gallery02.json');
        //isDemoGalleryLoaded.value = true;
    }
    sceneMode.value = 'gallery'
    hideGroup([analyseGroup, generateGroup]) //not sure if we need this
    //console.log('analyseGroup hidden');
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



const arrangeMeshesBehindImages = (
    userGalleryGroup: THREE.Group,
    generateGroup: THREE.Group,
    initialZOffset: number = -0.5, // Starting Z-offset
    incrementZOffset: number = -0.5 // Incremental Z-offset for additional meshes
) => {
    userGalleryGroup.children.forEach((imageObject) => {
        const imageName = imageObject.name; // Match by name or another identifier

        // Find all corresponding meshes for this image
        const correspondingMeshes = generateGroup.children.filter(
            (mesh) => mesh.userData.name === imageName
        );

        if (correspondingMeshes.length > 0) {
            correspondingMeshes.forEach((mesh, index) => {
                // Calculate Z-offset for this mesh
                const zOffset = initialZOffset + index * incrementZOffset;

                // Set position for the mesh
                mesh.position.set(
                    imageObject.position.x,
                    imageObject.position.y,
                    imageObject.position.z + zOffset
                );

                //console.log(`Positioning mesh "${imageName}" at:`, mesh.position);

                // Set position for edgesMesh to match the mesh
                if (mesh.userData.edgesMesh) {
                    const edgesMesh = mesh.userData.edgesMesh;
                    edgesMesh.position.copy(mesh.position);
                    //console.log(`Aligning edgesMesh for "${imageName}" at:`, edgesMesh.position);
                }
            });
        }
        // else {
        //     console.warn(`No corresponding meshes found for image "${imageName}"`);
        // }
    });
};





//even newer version with user specified gallery file
const loadCustomGallery = async (filePath: string) => {
    ///////////////should we empty generateGroup here? - i may regret this later
    //we have to decide if the user wants to keep the generated meshes or not when loading a new gallery
    //i will temorarily add the same thing to newEmptyGallery
    emptyGroup(generateGroup);
    /////////////////
    //console.log('load custom gallery');
    gallery.value = 'user';
    //await loadGalleryImages(filePath, userGalleryGroup, generateGroup);
    await loadGalleryImages(filePath, userGalleryGroup, generateGalleryGroup);
    //hideGroup([analyseGroup, generateGroup, demoGalleryGroup]);
    hideGroup([analyseGroup, demoGalleryGroup, generateDemoGalleryGroup]);
    //console.log('userGalleryGroup.children.length:', userGalleryGroup.children.length);
    sceneMode.value = 'gallery';
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


const downloadGallery = async (
    userGalleryGroup: THREE.Group,
    meshGroup: THREE.Group
) => {
    if (userGalleryGroup.children.length === 0) {
        alert("The user gallery is empty. There is nothing to download.");
        return;
    }

    // Step 1: Transfer relevant objects from generateGroup to meshGroup based on associated images
    if (generateGroup.children.length > 0) {
        generateGroup.children.forEach((child) => {
            const isAssociatedWithUserGallery = userGalleryGroup.children.some(
                (imageObject) => imageObject.userData.image.file_name === child.userData.name
            );

            if (isAssociatedWithUserGallery) {
                // Check if an object with identical parameters already exists in the meshGroup
                const isDuplicate = meshGroup.children.some((existingChild) => {
                    return (
                        existingChild.userData.name === child.userData.name &&
                        existingChild.userData.creationMethod === child.userData.creationMethod &&
                        existingChild.userData.createdAt === child.userData.createdAt
                    );
                });

                // Add the object only if it is not a duplicate
                if (!isDuplicate) {
                    meshGroup.add(child.clone()); // Clone to avoid modifying the original object
                    console.log(`Added unique object "${child.name}" to meshGroup.`);
                } else {
                    console.log(`Skipped duplicate object "${child.name}".`);
                }
            } else {
                console.log(`Skipped object "${child.name}" as its associated image is not in userGalleryGroup.`);
            }
        });
    }

    // Step 2: Serialize userGalleryGroup and associated meshes
    const galleryData = userGalleryGroup.children.map((imageObject) => {
        const imageFileName = imageObject.userData.image.file_name;

        // Serialize meshes associated with this image
        const associatedMeshes = meshGroup.children
            .filter((mesh) => mesh.userData.name === imageFileName)
            .filter((mesh, index, self) =>
                self.findIndex(
                    (m) =>
                        m.userData.name === mesh.userData.name &&
                        m.userData.creationMethod === mesh.userData.creationMethod &&
                        m.userData.createdAt === mesh.userData.createdAt
                ) === index // Ensure uniqueness in the associatedMeshes array
            )
            .map((mesh) => ({
                creationMethod: mesh.userData.creationMethod,
                createdAt: mesh.userData.createdAt,
                mesh: customSerializeMeshWithEdges(mesh as THREE.Mesh), // Streamlined serialization
            }));

        return {
            file_name: imageFileName,
            resolution: imageObject.userData.image.resolution,
            meshes: associatedMeshes,
        };
    });

    // Step 3: Convert to JSON and save
    const jsonData = JSON.stringify(galleryData, null, 2);
    const blob = new Blob([jsonData], { type: "application/json" });

    if ("showSaveFilePicker" in window) {
        try {
            const options = {
                suggestedName: "userGalleryData.json",
                types: [
                    {
                        description: "JSON Files",
                        accept: { "application/json": [".json"] },
                    },
                ],
            };
            const handle = await (window as any).showSaveFilePicker(options);
            const writable = await handle.createWritable();
            await writable.write(blob);
            await writable.close();
            console.log("Gallery data saved successfully.");
        } catch (error) {
            console.error("Error saving gallery data:", error);
        }
    } else {
        const defaultFileName = "userGalleryData.json";
        const fileName = prompt("Enter a filename for the gallery data:", defaultFileName);
        if (fileName) {
            const url = URL.createObjectURL(blob);
            const downloadLink = document.createElement("a");
            downloadLink.href = url;
            downloadLink.download = fileName;
            document.body.appendChild(downloadLink);
            downloadLink.click();
            document.body.removeChild(downloadLink);
            URL.revokeObjectURL(url);
        }
    }
};




const serializeMaterial = (material: THREE.Material): any => {
    const materialJson = material.toJSON();

    // Serialize envMapRotation as an array if it exists
    if (material instanceof THREE.MeshStandardMaterial && material.envMapRotation) {
        materialJson.envMapRotation = material.envMapRotation.toArray();
    }

    return materialJson;
};


const customSerializeMeshWithEdges = (mesh: THREE.Mesh): any => {
    const isMaterialArray = (material: THREE.Material | THREE.Material[]): material is THREE.Material[] =>
        Array.isArray(material);

    const serializedMesh = {
        type: mesh.type,
        geometry: mesh.geometry.toJSON(),
        material: isMaterialArray(mesh.material)
            ? mesh.material.map(serializeMaterial)
            : serializeMaterial(mesh.material),
        position: mesh.position.toArray(),
        rotation: mesh.rotation.toArray(),
        scale: mesh.scale.toArray(),
        userData: { ...mesh.userData }
    };

    if (mesh.userData.edgesMesh) {
        const edgesMesh = mesh.userData.edgesMesh;
        serializedMesh.userData.edgesMesh = {
            type: edgesMesh.type,
            geometry: edgesMesh.geometry.toJSON(),
            material: isMaterialArray(edgesMesh.material)
                ? edgesMesh.material.map(serializeMaterial)
                : serializeMaterial(edgesMesh.material),
            position: edgesMesh.position.toArray(),
            rotation: edgesMesh.rotation.toArray(),
            scale: edgesMesh.scale.toArray(),

            //////////////////// exclude the edgesMesh from raycasting
            userData: {
                ...edgesMesh.userData,
                hasCustomRaycast: true // Add flag for custom raycast
            }
            ////////////////////
        };
    }

    return serializedMesh;
};





const deserializeGeometry = (geometryJson: any): THREE.BufferGeometry | THREE.EdgesGeometry => {
    if (!geometryJson || !geometryJson.metadata) {
        throw new Error("Invalid geometry JSON data.");
    }

    //console.log("Geometry JSON:", geometryJson);

    // Handle EdgesGeometry
    if (geometryJson.type === "EdgesGeometry") {
        //console.log("Detected EdgesGeometry. Reconstructing...");
        // Deserialize the base BufferGeometry
        const baseGeometryJson = geometryJson.geometry;
        const baseGeometry = deserializeGeometry(baseGeometryJson); // Recursively handle base geometry

        // Create EdgesGeometry from the base geometry
        return new THREE.EdgesGeometry(baseGeometry, geometryJson.thresholdAngle);
    }

    // Handle BufferGeometry
    if (geometryJson.metadata.type === "BufferGeometry") {
        const loader = new THREE.BufferGeometryLoader();
        return loader.parse(geometryJson);
    }

    throw new Error(`Unsupported geometry type: ${geometryJson.type}`);
};


const deserializeMaterial = (materialJson: any): THREE.Material | THREE.Material[] => {
    if (!materialJson) {
        throw new Error("Invalid material JSON data.");
    }

    const loader = new THREE.MaterialLoader();
    try {
        if (Array.isArray(materialJson)) {
            return materialJson.flatMap((mat) => deserializeMaterial(mat));
        }

        const material = loader.parse(materialJson);

        // Handle envMapRotation if present
        if (materialJson.envMapRotation) {
            if (material instanceof THREE.MeshStandardMaterial) {
                material.envMapRotation = new THREE.Euler().fromArray(materialJson.envMapRotation);
            }
        }

        return material;
    } catch (error) {
        console.error("Error parsing material JSON:", error);
        throw error;
    }
};


const deserializeMeshWithEdges = (meshData: any): THREE.Mesh => {
    const geometry = deserializeGeometry(meshData.geometry);
    const material = deserializeMaterial(meshData.material);

    const mesh = new THREE.Mesh(geometry, material);
    mesh.position.fromArray(meshData.position);
    mesh.rotation.fromArray(meshData.rotation);
    mesh.scale.fromArray(meshData.scale);
    mesh.userData = meshData.userData;

    if (meshData.userData.edgesMesh) {
        const edgesMeshData = meshData.userData.edgesMesh;
        const edgesGeometry = deserializeGeometry(edgesMeshData.geometry);
        const edgesMaterial = deserializeMaterial(edgesMeshData.material);

        const edgesMesh = new THREE.LineSegments(edgesGeometry, edgesMaterial);
        edgesMesh.position.fromArray(edgesMeshData.position);
        edgesMesh.rotation.fromArray(edgesMeshData.rotation);
        edgesMesh.scale.fromArray(edgesMeshData.scale);

        ////////////////////////////////  exclude the edgesMesh from raycasting
        edgesMesh.userData = { ...edgesMeshData.userData };
        // Reapply the custom raycast method
        if (edgesMesh.userData.hasCustomRaycast) {
            edgesMesh.raycast = () => { };
        }
        ////////////////////////////////

        mesh.userData.edgesMesh = edgesMesh;
    }

    return mesh;
};





////////////////////////

const newEmptyGallery = () => {
    if (userGalleryGroup.children.length > 0) {
        const userConfirmed = confirm("Do you want to save the current user gallery before creating a new one?");
        if (userConfirmed) {
            downloadGallery(userGalleryGroup, generateGroup).then(() => {
                emptyGroup(userGalleryGroup);
                gallery.value = 'demo';
                sceneMode.value = 'analyse';
                hideGroup([analyseGroup, generateGroup, demoGalleryGroup]);
            });
        } else {
            emptyGroup(userGalleryGroup);
            emptyGroup(generateGroup); ///maybe remove this later
            emptyGroup(generateGalleryGroup); ///maybe remove this later
            gallery.value = 'demo';
            sceneMode.value = 'analyse';
            hideGroup([analyseGroup, generateGroup, demoGalleryGroup]);
        }
    } else {
        emptyGroup(userGalleryGroup);
        emptyGroup(generateGroup);///maybe remove this later
        emptyGroup(generateGalleryGroup);///maybe remove this later
        gallery.value = 'demo';
        sceneMode.value = 'analyse';
        hideGroup([analyseGroup, generateGroup, demoGalleryGroup]);
    }
};




const requestOllamaApi = async () => {
    fetch('https://ostrich-fleet-correctly.ngrok-free.app/api/generate', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            //'ngrok-skip-browser-warning': 'true'
        },
        body: JSON.stringify({
            //"model": "llama3.1:latest",
            "model": "llama2:7b",
            "temperature": 0,
            "prompt": "Tell me a joke!",
            "system": "You are an ironic Programmer.",
            "stream": false
        }),
    })
        .then(response => {
            if (response.status === 204) {
                console.warn("Received 204 No Content. Verify server behavior or request payload.");
                return null;
            }
            if (!response.ok) throw new Error(`HTTP error! Status: ${response.status}`);
            return response.json();
        })
        .then(data => {
            if (data) {
                console.log("Response Data:", data);
            } else {
                console.log("No content returned from the server.");
            }
        })
        .catch(err => console.error(err));

}


//////////// WEBSOCKET APPROACH FOR COMFYUI API
// function generateUUID() {
//     return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, (char) => {
//         const random = (Math.random() * 16) | 0;
//         const value = char === 'x' ? random : (random & 0x3) | 0x8;
//         return value.toString(16);
//     });
// }

// const clientId = ref(generateUUID()); // Function to generate a unique client ID
// //const socket = new WebSocket(`ws://127.0.0.1:8188/ws?clientId=${clientId}`);
// let socket: WebSocket | null = null; // WebSocket instance

// // Function to set up the WebSocket connection
// const setupWebSocket = () => {
//     socket = new WebSocket(`wss://ostrich-fleet-correctly.ngrok-free.app/ws?clientId=${clientId.value}`);
//     //socket = new WebSocket(`wss://ostrich-fleet-correctly.ngrok-free.app/ws`);
//     //console.log('socket:', socket);

//     socket.onopen = handleWebSocketOpen;
//     //socket.onmessage = (event) => handleWebSocketMessage(event.data);
//     socket.onmessage = (event: MessageEvent<string>) => {
//         handleWebSocketMessage(event);
//     };
//     socket.onerror = handleWebSocketError;
//     socket.onclose = handleWebSocketClose;
// };

// // WebSocket event handlers
// const handleWebSocketOpen = () => {
//     console.log('WebSocket connection established');
// };



// ///////////
// // Define the message structure
// interface WebSocketMessage {
//     type: string;
//     data: {
//         status?: {
//             exec_info: {
//                 queue_remaining: number;
//             };
//         };
//         sid: string;
//     };
// }

// // Type the WebSocket handler
// const handleWebSocketMessage = (event: MessageEvent<string>) => {
//     try {
//         const message: WebSocketMessage = JSON.parse(event.data);
//         //console.log('Parsed WebSocket message:', message);

//         if (message.type === 'status') {
//             console.log('Status message received:', message.data);
//         }
//         if (message.type === 'executing' && 'node' in message.data && message.data.node === null) {
//             console.log('Prompt execution completed:', message.data);
//             if ('prompt_id' in message.data) {
//                 const promptId = message.data.prompt_id as string;
//                 //console.log('Fetching generated image for prompt:', promptId);
//                 // Add the image to the Three.js scene
//                 addGeneratedImageToScene(promptId, scene);
//             }
//         }
//     } catch (err) {
//         console.error('Error parsing WebSocket message:', err, 'Raw data:', event.data);
//     }
// };




// const handleWebSocketError = (error: Event) => {
//     console.error('WebSocket error:', error);
// };

// const handleWebSocketClose = () => {
//     console.log('WebSocket connection closed');
// };

// // Function to clean up the WebSocket
// const cleanupWebSocket = () => {
//     if (socket) {
//         socket.close();
//         socket = null;
//     }
// };

///////////// new websocket dynamic handling approach
function generateUUID() {
    return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, (char) => {
        const random = (Math.random() * 16) | 0;
        const value = char === 'x' ? random : (random & 0x3) | 0x8;
        return value.toString(16);
    });
}

// Define the message structure
interface WebSocketMessage {
    type: string;
    data: {
        status?: {
            exec_info: {
                queue_remaining: number;
            };
        };
        sid: string;
    };
}

const clientId = ref(generateUUID()); // Function to generate a unique client ID
let socket: WebSocket | null = null; // WebSocket instance
let isConnecting = false; // To prevent duplicate connection attempts

// Function to set up the WebSocket connection
const setupWebSocket = () => {
    if (socket && (socket.readyState === WebSocket.OPEN || socket.readyState === WebSocket.CONNECTING)) {
        console.log('WebSocket is already connected or connecting');
        return;
    }

    console.log('Setting up WebSocket connection...');
    socket = new WebSocket(`wss://ostrich-fleet-correctly.ngrok-free.app/ws?clientId=${clientId.value}`);

    socket.onopen = handleWebSocketOpen;
    socket.onmessage = (event: MessageEvent<string>) => {
        handleWebSocketMessage(event);
    };
    socket.onerror = handleWebSocketError;
    socket.onclose = handleWebSocketClose;
};

// Ensure the WebSocket connection is active
const ensureWebSocketConnected = async (): Promise<void> => {
    if (socket && socket.readyState === WebSocket.OPEN) {
        console.log('WebSocket is already open');
        return;
    }

    if (isConnecting) {
        console.log('WebSocket is connecting, please wait...');
        return;
    }

    isConnecting = true;

    return new Promise((resolve, reject) => {
        setupWebSocket();

        const timeout = setTimeout(() => {
            isConnecting = false;
            reject(new Error('WebSocket connection timed out'));
        }, 5000); // Timeout after 5 seconds
        if (socket) {
            socket.onopen = () => {
                clearTimeout(timeout);
                console.log('WebSocket connection established');
                isConnecting = false;
                resolve();
            };

            socket.onerror = (error) => {
                clearTimeout(timeout);
                console.error('WebSocket connection error:', error);
                isConnecting = false;
                reject(error);
            };
        }
    });
};

// WebSocket event handlers
const handleWebSocketOpen = () => {
    console.log('WebSocket connection established');
};

const handleWebSocketMessage = (event: MessageEvent<string>) => {
    try {
        const message: WebSocketMessage = JSON.parse(event.data);

        if (message.type === 'status') {
            console.log('Status message received:', message.data);
        }
        if (message.type === 'executing' && 'node' in message.data && message.data.node === null) {
            console.log('Prompt execution completed:', message.data);
            if ('prompt_id' in message.data) {
                const promptId = message.data.prompt_id as string;
                // Add the image to the Three.js scene
                addGeneratedImageToScene(promptId, scene);
            }
        }
    } catch (err) {
        console.error('Error parsing WebSocket message:', err, 'Raw data:', event.data);
    }
};

const handleWebSocketError = (error: Event) => {
    console.error('WebSocket error:', error);
};

const handleWebSocketClose = () => {
    console.log('WebSocket connection closed');
    socket = null;
    isConnecting = false;
    ensureWebSocketConnected().catch(err => {
        console.error('Reconnection failed:', err);
    });
};

// Function to clean up the WebSocket
const cleanupWebSocket = () => {
    if (socket) {
        console.log('Cleaning up WebSocket connection');
        socket.close();
        socket = null;
    }
};

// // Function to send data to the WebSocket
// const sendDataToWebSocket = async (data: any) => {
//     try {
//         await ensureWebSocketConnected(); // Ensure WebSocket is connected before sending
//         if (socket && socket.readyState === WebSocket.OPEN) {
//             console.log('Sending data:', data);
//             socket.send(JSON.stringify(data));
//         } else {
//             console.error('WebSocket is not open, unable to send data');
//         }
//     } catch (error) {
//         console.error('Error ensuring WebSocket connection:', error);
//     }
// };


//////////////////////////
const requestComfyUIAPI = async () => {
    hideGroup([generateGroup, generateGalleryGroup]);
    let imagePosition = new THREE.Vector3();
    if (activeImage.value) {
        const imageName = activeImage.value.userData.image.file_name; //this will later serve to send specific prompts to the comfyUI API
        imagePosition = activeImage.value.position;
    }

    const comfySeed = Math.floor(Math.random() * 1000000000);
    //const textPrompt = document.getElementById('textPrompt') as HTMLInputElement;
    const textPrompt = "Generate a futuristic concrete architecture archigra";

    const comfyPromptXL =
    {
        "prompt": {
            "3": {
                "inputs": {
                    //"seed": 1081382006886948,
                    "seed": comfySeed,
                    "steps": 30,
                    "cfg": 5.45,
                    "sampler_name": "euler",
                    "scheduler": "sgm_uniform",
                    "denoise": 1,
                    "model": [
                        "4",
                        0
                    ],
                    "positive": [
                        "16",
                        0
                    ],
                    "negative": [
                        "40",
                        0
                    ],
                    "latent_image": [
                        "53",
                        0
                    ]
                },
                "class_type": "KSampler",
                "_meta": {
                    "title": "KSampler"
                }
            },
            "4": {
                "inputs": {
                    "ckpt_name": "sd3.5_large.safetensors"
                },
                "class_type": "CheckpointLoaderSimple",
                "_meta": {
                    "title": "Load Checkpoint"
                }
            },
            "8": {
                "inputs": {
                    "samples": [
                        "3",
                        0
                    ],
                    "vae": [
                        "4",
                        2
                    ]
                },
                "class_type": "VAEDecode",
                "_meta": {
                    "title": "VAE Decode"
                }
            },
            "9": {
                "inputs": {
                    "filename_prefix": "ComfyUI",
                    "images": [
                        "8",
                        0
                    ]
                },
                "class_type": "SaveImage",
                "_meta": {
                    "title": "Save Image"
                }
            },
            "16": {
                "inputs": {
                    //"text": "a nice dog playing in the park in the style of a jungle painting",
                    //"text": "a nice dog playing in the jungle in the style of henri rousseau",
                    "text": textPrompt,
                    "clip": [
                        "43",
                        0
                    ]
                },
                "class_type": "CLIPTextEncode",
                "_meta": {
                    "title": "Positive Prompt"
                }
            },
            "40": {
                "inputs": {
                    "text": "",
                    "clip": [
                        "43",
                        0
                    ]
                },
                "class_type": "CLIPTextEncode",
                "_meta": {
                    "title": "Negative Prompt"
                }
            },
            "41": {
                "inputs": {
                    "clip_name": "t5xxl_fp16.safetensors",
                    "type": "sd3"
                },
                "class_type": "CLIPLoader",
                "_meta": {
                    "title": "Load CLIP"
                }
            },
            "42": {
                "inputs": {
                    "clip_name1": "clip_l.safetensors",
                    "clip_name2": "clip_g.safetensors",
                    "type": "sd3"
                },
                "class_type": "DualCLIPLoader",
                "_meta": {
                    "title": "DualCLIPLoader"
                }
            },
            "43": {
                "inputs": {
                    "clip_name1": "clip_l.safetensors",
                    "clip_name2": "clip_g.safetensors",
                    "clip_name3": "t5xxl_fp16.safetensors"
                },
                "class_type": "TripleCLIPLoader",
                "_meta": {
                    "title": "TripleCLIPLoader"
                }
            },
            "53": {
                "inputs": {
                    "width": 1024,
                    "height": 1024,
                    //"width": 512,
                    //"height": 512,
                    "batch_size": 1
                },
                "class_type": "EmptySD3LatentImage",
                "_meta": {
                    "title": "EmptySD3LatentImage"
                }
            }
        },
        client_id: clientId.value, // Use the client ID associated with the WebSocket
    }



    const comfyPromptXLTurbo =
    {
        "prompt": {
            "5": {
                "inputs": {
                    //"seed": comfySeed,
                    "width": 512,
                    "height": 512,
                    "batch_size": 1
                },
                "class_type": "EmptyLatentImage",
                "_meta": {
                    "title": "Empty Latent Image"
                }
            },
            "6": {
                "inputs": {
                    //"text": "beautiful landscape scenery glass bottle with a galaxy inside cute fennec fox snow HDR sunset",
                    "text": textPrompt,
                    "clip": [
                        "20",
                        1
                    ]
                },
                "class_type": "CLIPTextEncode",
                "_meta": {
                    "title": "CLIP Text Encode (Prompt)"
                }
            },
            "7": {
                "inputs": {
                    "text": "text, watermark",
                    "clip": [
                        "20",
                        1
                    ]
                },
                "class_type": "CLIPTextEncode",
                "_meta": {
                    "title": "CLIP Text Encode (Prompt)"
                }
            },
            "8": {
                "inputs": {
                    "samples": [
                        "13",
                        0
                    ],
                    "vae": [
                        "20",
                        2
                    ]
                },
                "class_type": "VAEDecode",
                "_meta": {
                    "title": "VAE Decode"
                }
            },
            "13": {
                "inputs": {
                    "add_noise": true,
                    "noise_seed": comfySeed,
                    "cfg": 1,
                    "model": [
                        "20",
                        0
                    ],
                    "positive": [
                        "6",
                        0
                    ],
                    "negative": [
                        "7",
                        0
                    ],
                    "sampler": [
                        "14",
                        0
                    ],
                    "sigmas": [
                        "22",
                        0
                    ],
                    "latent_image": [
                        "5",
                        0
                    ]
                },
                "class_type": "SamplerCustom",
                "_meta": {
                    "title": "SamplerCustom"
                }
            },
            "14": {
                "inputs": {
                    "sampler_name": "euler_ancestral"
                },
                "class_type": "KSamplerSelect",
                "_meta": {
                    "title": "KSamplerSelect"
                }
            },
            "20": {
                "inputs": {
                    "ckpt_name": "sd_xl_turbo_1.0_fp16.safetensors"
                },
                "class_type": "CheckpointLoaderSimple",
                "_meta": {
                    "title": "Load Checkpoint"
                }
            },
            "22": {
                "inputs": {
                    "steps": 1,
                    "denoise": 1,
                    "model": [
                        "20",
                        0
                    ]
                },
                "class_type": "SDTurboScheduler",
                "_meta": {
                    "title": "SDTurboScheduler"
                }
            },
            "25": {
                "inputs": {
                    "images": [
                        "8",
                        0
                    ]
                },
                "class_type": "PreviewImage",
                "_meta": {
                    "title": "Preview Image"
                }
            }
        },
        client_id: clientId.value, // Use the client ID associated with the WebSocket
    }

    try {
        await ensureWebSocketConnected(); // Ensure WebSocket is connected before sending
        if (socket && socket.readyState === WebSocket.OPEN) {
            fetch('https://ostrich-fleet-correctly.ngrok-free.app/prompt', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify(comfyPromptXLTurbo),
            })
                .then(response => response.json())
                .then(data => {
                    console.log('Prompt submitted:', data);
                })
                .catch(err => console.error('Error submitting prompt:', err));
            //socket.send(JSON.stringify(comfyPromptXLTurbo)); //doesnt work delete ASAP
        } else {
            console.error('WebSocket is not open, unable to send data');
        }
    } catch (error) {
        console.error('Error ensuring WebSocket connection:', error);
    }
    //////////////////////
    // fetch('https://ostrich-fleet-correctly.ngrok-free.app/prompt', {
    //     method: 'POST',
    //     headers: {
    //         'Content-Type': 'application/json',
    //     },
    //     body: JSON.stringify(comfyPromptXLTurbo),
    // })
    //     .then(response => response.json())
    //     .then(data => {
    //         console.log('Prompt submitted:', data);
    //     })
    //     .catch(err => console.error('Error submitting prompt:', err));

    /////////////////////// Send the data via WebSocket
    // try {
    //         await sendDataToWebSocket(comfyPromptXLTurbo);

    //         console.log('Prompt submitted successfully');
    //     } catch(err) {
    //         console.error('Error sending prompt to ComfyUI via WebSocket:', err);
    //     }
    ////////////////////

}

///////////////
const fetchGeneratedImage = async (promptId: string): Promise<string> => {
    const historyUrl = `https://ostrich-fleet-correctly.ngrok-free.app/history/${promptId}`;
    console.log(`Fetching history for promptId: ${promptId} at URL: ${historyUrl}`);

    try {
        const response = await fetch(historyUrl, {
            headers: {
                'ngrok-skip-browser-warning': 'true', // Add this header to bypass the safety page
            },
        });

        // Log the response for debugging
        console.log('Response:', response);

        // Read and parse the JSON body
        const rawText = await response.text();
        //console.log('Raw Response Text:', rawText);

        const history = JSON.parse(rawText);
        //console.log('Parsed History Response:', history);

        const outputs = history[promptId]?.outputs;

        if (outputs) {
            for (const nodeId in outputs) {
                const nodeOutput = outputs[nodeId];
                if (nodeOutput.images && nodeOutput.images.length > 0) {
                    const image = nodeOutput.images[0];
                    const params = new URLSearchParams({
                        filename: image.filename,
                        subfolder: image.subfolder,
                        type: image.type,
                    });

                    const imageUrl = `https://ostrich-fleet-correctly.ngrok-free.app/view?${params.toString()}`;
                    console.log(`Generated image URL: ${imageUrl}`);
                    return imageUrl;
                }
            }
        }
    } catch (err) {
        console.error('Error fetching or parsing generated image:', err);
    }

    throw new Error('No image found for promptId');
};

//////////////
const fetchImageAndCreateTexture = async (imageUrl: string, scene: THREE.Scene) => {
    try {
        console.log('Fetching image:', imageUrl);

        // Fetch the image with the custom header
        const response = await fetch(imageUrl, {
            headers: {
                'ngrok-skip-browser-warning': 'true',
            },
        });

        if (!response.ok) {
            throw new Error(`Failed to fetch image: ${response.status} ${response.statusText}`);
        }

        const blob = await response.blob();
        //console.log('Fetched image blob:', blob);

        // Create an object URL from the blob
        const imageObjectUrl = URL.createObjectURL(blob);
        /////////////////////////////// this part is temporary and should later be handled by a remote server to keep track of the files for the gallery
        const saveImageAsFile = (imageBlob: Blob, filename: string) => {
            const url = URL.createObjectURL(imageBlob);
            const a = document.createElement('a');
            a.href = url;
            a.download = filename || 'image.png';
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
        };
        const name = activeImage.value ? activeImage.value.userData.image.file_name : `generated_image_${Date.now()}`; //name is activeImage value if it exists, otherwise a generated name
        // Save the fetched image as a file
        saveImageAsFile(blob, name+'.png');
        ///////////////////////////////
        
        // Pass the object URL to a custom function to create the texture
        //createTextureFromImage(imageObjectUrl, scene); //before downloading function
        createTextureFromImage(imageObjectUrl, scene, name); //with downloading function

        // Clean up the object URL after it's used
        // Revoke the object URL after the image has loaded
        // NOTE: This is now handled inside `createTextureFromImage`
        //URL.revokeObjectURL(imageObjectUrl);

    } catch (err) {
        console.error('Error fetching and creating texture:', err);
    }
};

const createTextureFromImage = (imageObjectUrl: string, scene: THREE.Scene, planeName: string) => {
    const image = new Image();

    image.onload = () => {
        console.log('Image loaded for texture creation:', image);

        // Create a texture from the loaded image
        const texture = new THREE.Texture(image);
        texture.needsUpdate = true;

        // Create a plane geometry and apply the texture
        const geometry = new THREE.PlaneGeometry(1, 1); // Adjust size as needed
        const material = new THREE.MeshBasicMaterial({ map: texture, side: THREE.DoubleSide });
        const plane = new THREE.Mesh(geometry, material);

        // Position the plane and add it to the scene
        plane.position.set(0, 0, 0.1); // Adjust position as needed
        scene.add(plane);

        //console.log('Image plane added to the scene');
        // Revoke the object URL now that the image is loaded
        URL.revokeObjectURL(imageObjectUrl);

        ////////// integrate the image into the generateGroup and maybe generateGalleryGroup
        plane.visible = true
        //plane.userData = { image }
        plane.userData.isImage = true;
        //plane.name = image.file_name;
        // Step 3: Assign attributes and userData to mesh and edgesMesh
        //const name = activeImage.value ? activeImage.value.userData.image.file_name : `generated_image_${Date.now()}`; //name is activeImage value if it exists, otherwise a generated name
        // i have removed the plane name from here and will give it to the function from where it is called, so that i can download the image from the blob with the same / correct name
        const timestamp = new Date().toISOString();
        // Assign attributes to mesh
        //plane.name = name;
        plane.name = planeName;
        plane.userData = {
            //name: name,
            name: planeName,
            creationMethod: "vision",
            createdAt: timestamp,
            isGeneratedImage: true,
        };
        //geometry.setAttribute('position', new THREE.BufferAttribute(vertices, 3));
        // Apply rotations and position adjustments
        if (activeImage.value) {
            const aspectRatio = activeImage.value?.scale.x;
            plane.position.x += 2.1 * aspectRatio;
        } else {
            plane.position.x += 1.9;
        }
        //plane.position.y -= 0.2;
        //analyseGroup.add(plane);
        // Step 4: Add to generateGroup
        generateGroup.add(plane);

        /////////////// this part is experimental and i am not sure if i should keep it 
        // this part is for automatic addition of the generated geometry to the userGalleryGroup
        // Check if the current image is in the userGalleryGroup
        if (activeImage.value) {
            userGalleryGroup.children.forEach((child) => {
                if (child.name === activeImage.value?.userData.image.file_name) {
                    putImageIntoUserGallery();
                }
            });
        }
        //console.log('plane:', plane);
        //activeImage.value = plane // Set the active image to the newly loaded image

        ////////////////
    };

    image.onerror = (err) => {
        console.error('Error loading image for texture creation:', err);
    };

    // Set the image source to the fetched object URL
    image.crossOrigin = 'anonymous'; // Set crossOrigin to allow external images
    image.src = imageObjectUrl;
};




const addGeneratedImageToScene = async (promptId: string, scene: THREE.Scene) => {
    try {
        // Fetch the generated image URL
        const imageUrl = await fetchGeneratedImage(promptId);

        //console.log('Fetched image URL:', imageUrl);

        // Load the image as a texture and add it to the scene
        //loadTexture(imageUrl, scene);
        fetchImageAndCreateTexture(imageUrl, scene);
    } catch (err) {
        console.error('Error adding generated image to scene:', err);
    }
};




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
        //console.log('All images have been loaded at least once.');
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
    // console.log('index:', index);
    // console.log('loadedImages[index]:', loadedImages[index]);
    // console.log('image.file_name:', image.file_name);

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

    //here we call the function that shows the analysis results for the image
    showSelectedAnalysisResults();
}

//showSelectedAnalysisResults function
const showSelectedAnalysisResults = () => {
    if (semanticLinesVisible.value) {
        showSemanticLines();
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
    if (semanticAnalysisVisible.value && sceneMode.value === 'analyse') {
        showSemanticAnalysis(false);
    }
    if (houghLinesVisible.value) {
        showSemanticAnalysis(false);
    }
    if (houghCirclesVisible.value) {
        showSemanticAnalysis(false);
    }
    if (shapeRecognitionVisible.value) {
        showSemanticAnalysis(false);
    }
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
    console.log('trying to put image into user gallery');
    if (activeImage.value) {
        const activeImageName = activeImage.value.userData.image.file_name;

        // Step 1: Check if the image is already in the userGalleryGroup
        const isImageInGallery = userGalleryGroup.children.some(
            (child) => child.name === activeImageName
        );

        if (isImageInGallery) {
            console.log("Image is already in the user gallery.");
        } else {
            // Add a cloned image to the userGalleryGroup
            const clonedImage = activeImage.value.clone();
            clonedImage.name = activeImageName; // Ensure the name is set
            userGalleryGroup.add(clonedImage);
            console.log(`Added image "${activeImageName}" to userGalleryGroup.`);
        }

        // Step 2: Find corresponding meshes in the generateGroup
        const correspondingMeshes = generateGroup.children.filter(
            (mesh) => mesh.userData.name === activeImageName
        );

        correspondingMeshes.forEach((mesh) => {
            // Step 3: Check for duplicates based on name, creationMethod, and createdAt
            //THIS SHOULD TECHNICALLY NOT BE NECESSARY, SINCE WE ARE CLONING THE MESHES anymore
            const isMeshInGallery = generateGalleryGroup.children.some((child) => {
                return (
                    child.userData.name === mesh.userData.name &&
                    child.userData.creationMethod === mesh.userData.creationMethod &&
                    child.userData.createdAt === mesh.userData.createdAt
                );
            });

            //working well - but will try to make without cloning below
            // if (!isMeshInGallery) {
            //     // Clone the mesh
            //     const clonedMesh = mesh.clone();

            //     // Clone and attach edgesMesh if it exists
            //     if (mesh.userData.edgesMesh) {
            //         const clonedEdgesMesh = mesh.userData.edgesMesh.clone();
            //         clonedMesh.userData.edgesMesh = clonedEdgesMesh; // Link the cloned edgesMesh
            //         generateGalleryGroup.add(clonedEdgesMesh); // Add edgesMesh to the group
            //     }

            //     // Add the cloned mesh to the generateGalleryGroup
            //     generateGalleryGroup.add(clonedMesh);
            //     console.log(
            //         `Added mesh "${mesh.name}" with creationMethod "${mesh.userData.creationMethod}" to generateGalleryGroup.`
            //     );
            // } else {
            //     console.log(
            //         `Mesh "${mesh.name}" with the same creationMethod and createdAt is already in generateGalleryGroup.`
            //     );
            // }

            //trying to put mesh directly into generateGalleryGroup without cloning
            if (!isMeshInGallery) {
                // Add the mesh plues edgesMesh to the generateGalleryGroup
                generateGalleryGroup.add(mesh);
                if (mesh.userData.edgesMesh) {
                    generateGalleryGroup.add(mesh.userData.edgesMesh);
                }
                console.log(
                    `Added mesh "${mesh.name}" with creationMethod "${mesh.userData.creationMethod}" to generateGalleryGroup.`
                );
            } else {
                console.log(
                    `Mesh "${mesh.name}" with the same creationMethod and createdAt is already in generateGalleryGroup.`
                );
            }
        });

        // Step 4: Update gallery view
        gallery.value = 'user';
    } else {
        console.log("No active image to add to the user gallery.");
    }
}


// function addLikedObjectsToGallery(
//     activeImageName: string,
//     sourceGroup: THREE.Group,
//     targetGroup: THREE.Group
// ): void {
//     // Find all objects in sourceGroup that match the activeImageName
//     const matchingObjects = sourceGroup.children.filter(
//         (object) => object.name === activeImageName || object.name === `${activeImageName}_edges`
//     );

//     matchingObjects.forEach((object) => {
//         // Check if the object is already in the targetGroup
//         const isAlreadyInGallery = targetGroup.children.some(
//             (child) => child.name === object.name
//         );

//         if (!isAlreadyInGallery) {
//             // Clone the object and add it to the targetGroup
//             const clonedObject = object.clone();

//             // Optionally, clone the edgesMesh reference for the Mesh
//             if (object.userData.edgesMesh) {
//                 clonedObject.userData.edgesMesh = object.userData.edgesMesh.clone();
//             }

//             targetGroup.add(clonedObject);
//         }
//     });
// }



//lets make a function that resets the states for the analysis results
//this function should be called when we switch to generate mode - we will call it in the wather function for sceneMode
const resetAnalysisResults = () => {
    semanticLinesVisible.value = false;
    segmentationVisible.value = false;
    paintingSequenceVisible.value = false;
    imageDepthVisible.value = false;
    semanticAnalysisVisible.value = false;
    houghLinesVisible.value = false;
    houghCirclesVisible.value = false;
    shapeRecognitionVisible.value = false;
    contentSemanticAnalysis.value = ""; // Update the ref's value
    //stopTyping(); // Stop the typewriter effect - not sure if it shoud be here
}

const loadAnalyse = () => {
    hideGroup([demoGalleryGroup, userGalleryGroup, generateGroup, generateGalleryGroup])
    stopTyping(); // Stop the typewriter effect //am not sure - if this is good here - there seems to be a lot of redundancy, especially around that typewriter effect


    if (activeImage.value) {
        //console.log('am in analyse mode: ', activeImage.value); //to be deleted later
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

        moveCameraToPosition(activeImage.value.position)
    }

    else {
        console.log('no active image value - loading new random image'); //to be deleted later
        loadRandomNewImage();
    }
}

const lastCameraPosition = new THREE.Vector3(0, 0, 5) //set initial camera position has to be reconsidered - but still better than leaving it empty, otherwise zooms into nothing
const lastCameraTarget = new THREE.Vector3(0, 0, 0)
const moveCameraToPosition = (position: THREE.Vector3) => {
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

// const onClick = (e: Event) => {
//     const mouse = new THREE.Vector2();
//     mouse.x = (e as MouseEvent).clientX / window.innerWidth * 2 - 1;
//     mouse.y = -(e as MouseEvent).clientY / window.innerHeight * 2 + 1;
//     raycaster.setFromCamera(mouse, camera);

//     //first filter visible objects before raycasting, otherwise it gave some erratic behaviour
//     const visibleObjects = [...demoGalleryGroup.children, ...userGalleryGroup.children].filter(obj => obj.visible);
//     const intersects = raycaster.intersectObjects(visibleObjects);

//     //const intersects = raycaster.intersectObjects([...demoGalleryGroup.children, ...userGalleryGroup.children]);
//     if (intersects.length > 0 && !isDragging.value) {
//         const object = intersects[0].object;
//         //if (object.userData.isImage && object.visible) {  //maybe  we can make do without the visible check? YES delete this asap
//         //in future we should try to toggle the raycasting on and off for the different groups - so that we can avoid the visible check
//         //should use raycast = null for the groups we do not want to raycast
//         if (object.userData.isImage) {
//             activeImage.value = object
//             //console.log('object: ', object);
//             if (gallery.value === 'user') {
//                 emptyGroup(analyseGroup);
//             }
//             sceneMode.value = 'analyse'
//         }
//     }
// }

//updated onClick function to enter generateGallery mode, if the active image has associated objects in generateGalleryGroup
const onClick = (e: Event) => {
    const mouse = new THREE.Vector2();
    mouse.x = (e as MouseEvent).clientX / window.innerWidth * 2 - 1;
    mouse.y = -(e as MouseEvent).clientY / window.innerHeight * 2 + 1;
    raycaster.setFromCamera(mouse, camera);

    // Filter visible objects for raycasting
    const visibleObjects = [...demoGalleryGroup.children, ...userGalleryGroup.children, ...generateGalleryGroup.children].filter(obj => obj.visible);
    const intersects = raycaster.intersectObjects(visibleObjects);

    if (intersects.length > 0 && !isDragging.value) {
        const object = intersects[0].object;
        console.log('object: ', object);

        if (object.userData.isImage) {
            activeImage.value = object;
            // Check if the activeImage has associated objects in generateGalleryGroup
            const associatedObjects = generateGalleryGroup.children.filter(
                (child) => child.userData.name === activeImage.value?.userData.image.file_name
            );

            if (associatedObjects.length > 0) {
                //console.log(`Switching to generateGallery mode. Associated objects: ${associatedObjects.length}`);
                emptyGroup(analyseGroup); //this is just temporary - here we should test if we are already in generateGallery mode
                //and if so - we should test, if the clicked object is the original image, then go to analyse mode, or a generated result,
                // then we should go to generate mode
                //anyway - we need to empty the analysegroup everytime we come from any of the gallery modes, since the latest added images 
                //are rearranged - so we need to reload them at the new position
                if (sceneMode.value === 'generateGallery') {
                    sceneMode.value = 'analyse';
                }
                else {
                    sceneMode.value = 'generateGallery';
                }
            } else {
                //console.log('No associated objects found. Switching to analyse mode.');
                if (gallery.value === 'user') {
                    emptyGroup(analyseGroup);
                }
                sceneMode.value = 'analyse';
            }
        }

        if (object.userData.isGeneratedMesh) {
            console.log('object is a generated mesh');
            //no need to empty the analyseGroup here, since we only take action when we are in generateGallery mode
            //and thus must have already emptied the analyseGroup
            if (sceneMode.value === 'generateGallery') {
                sceneMode.value = 'generate';
            }
        }

    }
};




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

const semanticLinePoints: { [key: string]: number[][] } = {}; // Stores original points by image name- we have to think about adding these to semanticLineGroup

const showSemanticLines = async () => {
    let semanticLineGroup = scene.getObjectByName('semanticLineGroup_' + activeImage.value?.userData.image.file_name);
    // console.log('semanticLineGroup: ', semanticLineGroup);
    // console.log('analyseGroup: ', analyseGroup);
    if (semanticLineGroup) {
        // console.log('semantic lines found');
        // console.log('semanticLineGroup.visible: ', semanticLineGroup.visible);
        semanticLineGroup.visible = !semanticLineGroup.visible
        semanticLinesVisible.value = semanticLineGroup.visible
        // console.log('semanticLineGroup.visible: ', semanticLineGroup.visible);
        return;
    }
    else {
        semanticLineGroup = new THREE.Group();
        semanticLineGroup.name = 'semanticLineGroup_' + activeImage.value?.userData.image.file_name;
        analyseGroup.add(semanticLineGroup);
        semanticLinesVisible.value = true


        //////////////////////////////
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

        // // Initialize the storage for this image's line points if it doesn't exist
        // const imageName = activeImage.value?.userData.image.file_name;
        // if (!semanticLinePoints[imageName]) {
        //     semanticLinePoints[imageName] = [];
        //     // Store the original points for this line under the image name
        //     points.forEach((line: number[]) => {
        //         semanticLinePoints[imageName].push(line);
        //     });
        // }
        //lets always store the images points, so that they are updated when the position of the activeimage changes
        const imageName = activeImage.value?.userData.image.file_name;
        semanticLinePoints[imageName] = [];
        // Store the original points for this line under the image name
        points.forEach((line: number[]) => {
            semanticLinePoints[imageName].push(line);
        });

        points.forEach((line: number[]) => {
            const lineObject = addLine2(line);
            if (semanticLineGroup) {
                semanticLineGroup.add(lineObject);
            }

            // // Store the original points for this line under the image name
            // semanticLinePoints[imageName].push(line);

        });
        /////////////////////////////////////
        //we moved basically all the stuff into the condition above - so that we only load the semantic lines, if they are not already loaded

    }

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

    //const folderName = 'sam_out01_contours_10k_orig01_cleanup_merged_100/'
    const folderName = 'sam_out01_contours_10k_orig01_cleanup_merged/'
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
        //debugger

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

    //const folderName = 'learn2paint_out01_contours_cleanup_merged_100/'
    const folderName = 'learn2paint_out01_contours_cleanup_merged/'
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


const loadGenerate = () => {
    sceneMode.value = 'generate'
    //console.log('generateGroup:', generateGroup);
    hideGroup([demoGalleryGroup, userGalleryGroup, analyseGroup, generateGalleryGroup])
    //do we actually need to hide the analyseGroup? 
    //technically we should not need to hide it, since we want to see the analysis results of the image
    //however if we hide it and then show stuff selectively, we can control the visibility of the analysis results individually
    // first we will check, which image has been given to the generateGroup from the analyseGroup - on the basis of which new images and geometry will be generated

    if (activeImage.value) {
        //console.log('am in generate mode: ', activeImage.value); //to be deleted later
        //console.log('userGallerGroup.length', userGalleryGroup.children.length);
        let found = false;

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

        toggleObjectsByName(generateGroup, activeImage.value.userData.image.file_name, true);

        //moveCameraToPosition(activeImage.value.position)
        moveCameraToGeneratePosition(activeImage.value.position);
        showSelectedAnalysisResults();// later we have to differentiate between stuff that is shown in analyse mode and stuff that is shown in generate mode
        // - because why are we otherwise hiding the analyseGroup
        // i think this is not solved very elegantly
        //REMEMBER: rework later both functions:  resetAnalysisResults and showSelectedAnalysisResults
    }

    else {
        //console.log('no active image value - loading new random image'); //to be deleted later
        loadRandomNewImage(); // this is highly unlikely to happen
    }


}

//create function to zoom the camera to a specific position in relation to the activeImage and its width and height
//this function should be called when the generate mode is activated
const moveCameraToGeneratePosition = (position: THREE.Vector3) => {
    const targetPosition = position.clone()
    const targetLookat = position.clone()
    const currentLookat = controls.target.clone()
    // the targetPosition.z should be calculated based on the width and height of the activeImage

    const aspectRatio = activeImage.value?.scale.x
    if (aspectRatio === undefined) {
        return
    }
    targetPosition.z += 3.5 * aspectRatio
    targetPosition.x += 2 * aspectRatio
    targetLookat.x += 2 * aspectRatio

    tweenToPosition(targetPosition)
    tweenToCameraTarget(currentLookat, targetLookat)
}

type PointData = {
    x: number;
    y: number;
    z: number;
};

type MeshData = {
    vertices: number[];
    faces: number[];
};

// Function to send the points to Grasshopper for processing
async function sendPointsToGrasshopper(points: PointData[], imageSizeX: number, imageSizeY: number, imagePosition: THREE.Vector3, workflow: string): Promise<string | null> {
    try {
        const response = await fetch('http://localhost:5000', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            //body: JSON.stringify({ points }),
            body: JSON.stringify({ points, imageSizeX, imageSizeY, imagePosition, workflow }),
        });
        //how can i console log the json string here?
        // console.log('JSON.stringify({ points, imageSizeX, imageSizeY, imagePosition }):', JSON.stringify({ points, imageSizeX, imageSizeY, imagePosition, workflow }));
        // console.log('response:', response);
        if (!response.ok) {
            throw new Error('Network response was not ok ' + response.statusText);
        }

        const data = await response.text();
        const requestIdMatch = data.match(/Request ID: ([a-z0-9-]+)/i);
        const requestId = requestIdMatch ? requestIdMatch[1] : null;
        //console.log('Request ID:', requestId);

        return requestId;
    } catch (error) {
        console.error('Error in sendPointsToGrasshopper:', error);
        return null;
    }
}


async function fetchProcessedMesh(requestId: string): Promise<MeshData | null> {
    try {
        const response = await fetch(`http://localhost:5000?id=${requestId}`, {
            method: 'GET',
            headers: {
                'Content-Type': 'application/json',
            },
        });

        if (!response.ok) {
            throw new Error('Network response was not ok ' + response.statusText);
        }

        const meshData: MeshData = await response.json();
        //console.log('Received processed mesh:', meshData);

        return meshData;
    } catch (error) {
        console.error('Error in fetchProcessedMesh:', error);
        return null;
    }
}


//revised function to create mesh from data
function createMeshFromData(meshData: MeshData, workflow: string): THREE.Mesh | null {
    if (!meshData.vertices) {
        console.error('Invalid mesh data');
        return null;
    }

    // Step 1: Initialize BufferGeometry and set vertices and faces
    const geometry = new THREE.BufferGeometry();
    const vertices = new Float32Array(meshData.vertices);
    const faces = new Uint32Array(meshData.faces);
    geometry.setIndex(new THREE.BufferAttribute(faces, 1));
    geometry.setAttribute('position', new THREE.BufferAttribute(vertices, 3));

    // Compute normals for correct lighting response
    geometry.computeVertexNormals();

    // Use a material with lighting properties
    const material = new THREE.MeshPhongMaterial({ color: 0xFFFFFF, side: THREE.DoubleSide });
    const mesh = new THREE.Mesh(geometry, material);

    // Center the geometry inside the mesh
    mesh.geometry.computeBoundingBox();
    const center = new THREE.Vector3();
    if (mesh.geometry.boundingBox) {
        mesh.geometry.boundingBox.getCenter(center);
    }
    mesh.geometry.center();
    mesh.position.add(center);

    // Apply rotations and position adjustments
    mesh.rotation.x = -Math.PI * 0.2;
    mesh.rotation.z = Math.PI * 0.2;
    if (activeImage.value) {
        const aspectRatio = activeImage.value?.scale.x;
        mesh.position.x += 2.1 * aspectRatio;
    } else {
        mesh.position.x += 1.9;
    }
    mesh.position.y -= 0.2;

    // Step 2: Create edgesMesh
    const edgesGeometry = new THREE.EdgesGeometry(geometry);
    const lineMaterial = new THREE.LineBasicMaterial({ color: 0x000000 }); // Customize color as needed
    const edgesMesh = new THREE.LineSegments(edgesGeometry, lineMaterial);

    // Apply the same transformations to the edgesMesh
    edgesMesh.position.copy(mesh.position);
    edgesMesh.rotation.copy(mesh.rotation);
    edgesMesh.scale.copy(mesh.scale);
    edgesMesh.scale.multiplyScalar(1.001); // Avoid z-fighting
    edgesMesh.raycast = () => { }; // Disable raycasting for edgesMesh - let me see, if this works
    edgesMesh.userData.customRaycast = true; // Mark the edgesMesh as having a custom raycast


    // Step 3: Assign attributes and userData to mesh and edgesMesh
    const name = activeImage.value ? activeImage.value.userData.image.file_name : `mesh_${Date.now()}`;
    const timestamp = new Date().toISOString();

    // Assign attributes to mesh
    mesh.name = name;
    mesh.userData = {
        edgesMesh: edgesMesh, // Direct runtime reference
        name: name,
        creationMethod: workflow,
        createdAt: timestamp,
        isGeneratedMesh: true,
    };

    // Assign attributes to edgesMesh
    edgesMesh.name = `${name}_edges`;
    edgesMesh.userData = {
        parentMeshName: name, // Link to parent mesh by name
        creationMethod: workflow,
        createdAt: timestamp,
    };

    // Step 4: Add to generateGroup
    generateGroup.add(mesh);
    generateGroup.add(edgesMesh);

    /////////////// this part is experimental and i am not sure if i should keep it 
    // this part is for automatic addition of the generated geometry to the userGalleryGroup
    // Check if the current image is in the userGalleryGroup
    if (activeImage.value) {
        userGalleryGroup.children.forEach((child) => {
            if (child.name === activeImage.value?.userData.image.file_name) {
                putImageIntoUserGallery();
            }
        });
    }


    return mesh;
}


// Function to generate a surface relief mesh from the semantic line points
async function processAndDisplayMesh(points: PointData[], imageSizeX: number, imageSizeY: number, imagePosition: THREE.Vector3, workflow: string) {
    //const requestId = await sendPointsToGrasshopper(points);
    const requestId = await sendPointsToGrasshopper(points, imageSizeX, imageSizeY, imagePosition, workflow);
    if (!requestId) {
        console.error('Failed to get request ID');
        return;
    }

    // Wait for the mesh to be processed in Grasshopper
    const intervalId = setInterval(async () => {
        const meshData = await fetchProcessedMesh(requestId);
        if (meshData) {
            createMeshFromData(meshData, workflow);
            clearInterval(intervalId); // Clear the interval once the mesh is fetched and created
        }
    }, 100); // Adjust interval as necessary }




}


const generateSurfaceRelief = async (workflow: string) => {
    //we should probably hide the generateGroup here first to hide the previously generated geometry
    hideGroup([generateGroup, generateGalleryGroup]);

    let points: PointData[] = [];
    let imageSizeX = 0;
    let imageSizeY = 0;
    let imagePosition = new THREE.Vector3();
    if (activeImage.value) {
        const imageName = activeImage.value.userData.image.file_name;
        imageSizeX = activeImage.value.userData.image.resolution.width;
        imageSizeY = activeImage.value.userData.image.resolution.height;
        imagePosition = activeImage.value.position;
        //console.log('semanticLinePoints:', semanticLinePoints);
        const linesData = semanticLinePoints[imageName];
        //console.log('linesData:', linesData);
        if (linesData) {
            linesData.forEach(lineData => {
                for (let i = 0; i < lineData.length; i += 3) {
                    points.push({ x: lineData[i], y: lineData[i + 1], z: lineData[i + 2] });
                }
                //console.log('Points:', points);
                //processAndDisplayMesh(points);

            });
        } else {
            //console.log('No lines found for this image.');
            points = [
                { x: 522, y: 2, z: 3 },
                { x: 4, y: 5, z: 6 },
                { x: 7, y: 8, z: 9 }
            ];
        }

    }

    // //processAndDisplayMesh(points);
    processAndDisplayMesh(points, imageSizeX, imageSizeY, imagePosition, workflow);

}


/////////// below here - trying to implement a force directed graph layout for the generated objects associated with the activeImage
// Constants for the force simulation
// for older versions of the constants check the older file versions e.g. Mainpage97.vue

//VERSION A - still very good, spreads out more, is also a bit slower
// new constants trying to tie the graph more together
//this gives a nice spiralling helix like structure, with long strands
// const springConstant = 0.1;
// const restLength = 1;
// const repulsionStrength = 0.3;
// const dampingFactor = 0.6;
// const maxVelocity = 0.05;
// const minVelocityThreshold = 0.04;
// const stabilizationFrames = 60;
// /////////

//VERSION B - better for now, much narrower spreading, plus a bit faster
//this gives a more linear structure, with shorter strands
// new constants trying to tie the graph even more together
const springConstant = 0.3;
const restLength = 0.1;
const repulsionStrength = 0.3;
const dampingFactor = 0.6;
const maxVelocity = 0.05;
const minVelocityThreshold = 0.04;
const stabilizationFrames = 60;


// Variables for the force simulation
let stabilizedFrames = 0;
let graphStabilized = false;

let nodes: { id: string, mesh: THREE.Object3D, edgesMesh?: THREE.Object3D, helper?: THREE.Object3D, velocity: THREE.Vector3, central?: boolean, initialPosition?: THREE.Vector3 }[] = []; // Array for iteration
let nodeMap = new Map<string, { mesh: THREE.Object3D, edgesMesh?: THREE.Object3D, helper?: THREE.Object3D, velocity: THREE.Vector3, central?: boolean }>(); // Map for quick lookups
let edges: THREE.LineSegments | null = null; // LineSegments object for edges
let graphData: { nodes: any[], links: any[] } | null = null; // Graph data (nodes and links)
let edgePositions: Float32Array | null = null; // Float32Array for edge positions
let edgeGeometry: THREE.BufferGeometry | null = null; // BufferGeometry for edges

const generateUniqueId = (obj: THREE.Object3D): string => {
    return `${obj.userData.name}_${obj.userData.creationMethod}_${obj.userData.createdAt}`;
};


/////////

function createGraphDataFromGallery(activeImage: THREE.Object3D, associatedObjects: THREE.Object3D[]) {
    const graphData: {
        nodes: { id: string; group: number }[],
        links: { source: string; target: string; value: number }[]
    } = {
        nodes: [],
        links: []
    };

    // Add activeImage as the central node
    graphData.nodes.push({ id: activeImage.name, group: 1 });

    // Group objects by creationMethod
    const groupedByCreationMethod: { [method: string]: THREE.Object3D[] } = {};
    associatedObjects.forEach((obj) => {
        const creationMethod = obj.userData.creationMethod;
        if (!groupedByCreationMethod[creationMethod]) {
            groupedByCreationMethod[creationMethod] = [];
        }
        groupedByCreationMethod[creationMethod].push(obj);
    });

    // Process each creationMethod group
    Object.keys(groupedByCreationMethod).forEach((method, groupIndex) => {
        const objects = groupedByCreationMethod[method];

        // Sort objects in chronological order based on createdAt
        objects.sort((a, b) => new Date(a.userData.createdAt).getTime() - new Date(b.userData.createdAt).getTime());

        // Add nodes for this creationMethod
        objects.forEach((obj) => {
            const uniqueId = generateUniqueId(obj); // Generate a unique ID
            graphData.nodes.push({ id: uniqueId, group: groupIndex + 2 }); // Different group for each creationMethod
        });

        // Connect the first object to the central image
        if (objects.length > 0) {
            const firstId = generateUniqueId(objects[0]);
            graphData.links.push({ source: activeImage.name, target: firstId, value: 1 });
        }

        // Connect each object to the next one in chronological order
        for (let i = 0; i < objects.length - 1; i++) {
            const sourceId = generateUniqueId(objects[i]);
            const targetId = generateUniqueId(objects[i + 1]);
            graphData.links.push({ source: sourceId, target: targetId, value: 1 });
        }
    });

    // Avoid duplicate links by ensuring uniqueness in links
    const uniqueLinks = new Set<string>();
    graphData.links = graphData.links.filter((link) => {
        const linkKey = `${link.source}-${link.target}`;
        if (uniqueLinks.has(linkKey)) {
            return false; // Skip duplicate link
        }
        uniqueLinks.add(linkKey);
        return true;
    });
    return graphData;
}

function setupForceGraphFromGallery(activeImage: THREE.Object3D, generateGalleryGroup: THREE.Group) {
    // Get associated objects for the activeImage
    const associatedObjects = generateGalleryGroup.children.filter(
        (child) => child.userData.name === activeImage.userData.image.file_name
    );

    // Build graph data with unique IDs
    graphData = createGraphDataFromGallery(activeImage, associatedObjects);

    // Initialize nodes and nodeMap
    nodes = [];
    nodeMap = new Map();

    // Add nodes to the scene and nodeMap
    //////////
    graphData.nodes.forEach((nodeData) => {
        let associatedMesh = associatedObjects.find((obj) => generateUniqueId(obj) === nodeData.id);

        // Check if the node is the central image
        if (nodeData.id === activeImage.userData.image.file_name) {
            nodeData.central = true; // Mark as central without modifying id
            //console.log('Central node:', nodeData.id);
        }

        if (associatedMesh) {
            // Get world position and rotation of the associatedMesh
            const worldPosition = new THREE.Vector3();
            const worldRotation = new THREE.Quaternion();

            associatedMesh.getWorldPosition(worldPosition);
            associatedMesh.getWorldQuaternion(worldRotation);

            //add a very slight random offset to the position
            associatedMesh.position.set(worldPosition.x + (Math.random() * 0.1 - 0.05), worldPosition.y + (Math.random() * 0.1 - 0.05), worldPosition.z + (Math.random() * 0.1 - 0.05));

            // Track both mesh and edgesMesh as one unit in the nodeMap
            const nodeEntry = {
                id: nodeData.id,
                mesh: associatedMesh,
                edgesMesh: associatedMesh.userData.edgesMesh,
                velocity: new THREE.Vector3(),
            };
            nodes.push(nodeEntry);
            nodeMap.set(nodeData.id, nodeEntry);

        } else {
            // Handle central image or fallback
            const defaultEntry = { id: nodeData.id, mesh: activeImage, velocity: new THREE.Vector3(), central: nodeData.central };
            //console.log('defaultEntry:', defaultEntry);
            nodes.push(defaultEntry);
            nodeMap.set(nodeData.id, defaultEntry);
        }
    });

    // Initialize edges
    const edgeMaterial = new THREE.LineBasicMaterial({ color: 0xffffff });
    edgePositions = new Float32Array(graphData.links.length * 2 * 3); // 2 points per line
    edgeGeometry = new THREE.BufferGeometry();
    edgeGeometry.setAttribute('position', new THREE.BufferAttribute(edgePositions, 3));
    edges = new THREE.LineSegments(edgeGeometry, edgeMaterial);
    //scene.add(edges); //no need to call this if we add the edges to the linesForceGraphGroup
    linesForceGraphGroup.add(edges);

    // Set initial edge positions
    graphData.links.forEach((link, index) => {
        const sourceNode = nodeMap.get(link.source)?.mesh;
        const targetNode = nodeMap.get(link.target)?.mesh;

        if (sourceNode && targetNode) {
            const startIdx = index * 6;
            edgePositions?.set(
                [sourceNode.position.x, sourceNode.position.y, sourceNode.position.z,
                targetNode.position.x, targetNode.position.y, targetNode.position.z], startIdx
            );
            //console.log(`edgePositions[${index}]:`, edgePositions?.slice(startIdx, startIdx + 6));
        }
    });

    console.log("Force graph initialized with actual scene positions.");

}

function updateSpringForces() {
    if (graphData) {
        graphData.links.forEach((link) => {
            const sourceNode = nodeMap.get(link.source);
            const targetNode = nodeMap.get(link.target);

            if (sourceNode && targetNode) {
                const direction = new THREE.Vector3().subVectors(targetNode.mesh.position, sourceNode.mesh.position);
                const distance = direction.length();
                const displacement = distance - restLength;
                const forceMagnitude = springConstant * displacement;
                const force = direction.normalize().multiplyScalar(forceMagnitude);

                sourceNode.velocity.add(force);
                targetNode.velocity.sub(force);
            }
        });
    }
}

function updateRepulsiveForces() {
    nodes.forEach((nodeA, i) => {
        for (let j = i + 1; j < nodes.length; j++) {
            const nodeB = nodes[j];
            const direction = new THREE.Vector3().subVectors(nodeB.mesh.position, nodeA.mesh.position);
            const distance = direction.length() + 0.01;
            const forceMagnitude = repulsionStrength / (distance * distance);
            const force = direction.normalize().multiplyScalar(forceMagnitude);
            nodeA.velocity.sub(force);
            nodeB.velocity.add(force);
        }
    });
}

function updateEdges() {
    if (graphData) {
        graphData.links.forEach((link, index) => {
            const sourceNode = nodeMap.get(link.source)?.mesh;
            const targetNode = nodeMap.get(link.target)?.mesh;

            if (sourceNode && targetNode) {
                const sourceWorldPos = new THREE.Vector3();
                const targetWorldPos = new THREE.Vector3();
                sourceNode.getWorldPosition(sourceWorldPos);
                targetNode.getWorldPosition(targetWorldPos);
                const startIdx = index * 6;
                edgePositions?.set(
                    [sourceWorldPos.x, sourceWorldPos.y, sourceWorldPos.z,
                    targetWorldPos.x, targetWorldPos.y, targetWorldPos.z], startIdx
                );

            } else {
                console.warn(`Missing node for link: ${link.source} or ${link.target}`);
            }
        });
    }

    if (edgeGeometry) {
        edgeGeometry.attributes.position.needsUpdate = true;
    }
}

function applyDampingAndCapVelocity() {
    nodes.forEach((node) => {
        node.velocity.multiplyScalar(dampingFactor);
        if (node.velocity.length() > maxVelocity) {
            node.velocity.setLength(maxVelocity);
        }
    });
}

//// well working updatePositions function, including moving the central node
// function updatePositions() {
//     nodes.forEach((node) => {
//         // Update mesh position
//         node.mesh.position.add(node.velocity);

//         // Update edgesMesh position if it exists
//         if (node.edgesMesh) {
//             node.edgesMesh.position.copy(node.mesh.position);
//         }

//         // // Update helper position for debugging
//         // if (node.helper) {
//         //     node.helper.position.copy(node.mesh.position);
//         // }
//     });
// }

//this should exclude the central node
function updatePositions() {
    nodes.forEach((node) => {
        if (node.central) {
            node.velocity.set(0, 0, 0); // Reset velocity for the central node
            return;
        }

        node.mesh.position.add(node.velocity);

        if (node.edgesMesh) {
            node.edgesMesh.position.copy(node.mesh.position);
        }

    });
}


//this is not required now, but we might need it later
// function updateRotations() {
//     nodes.forEach((node) => {
//         // Example: Apply some rotation logic if needed
//         const rotationDelta = new THREE.Euler(0, 0.01, 0);
//         node.mesh.rotation.x += rotationDelta.x;
//         node.mesh.rotation.y += rotationDelta.y;
//         node.mesh.rotation.z += rotationDelta.z;

//         // Synchronize edgesMesh rotation if it exists
//         if (node.edgesMesh) {
//             node.edgesMesh.rotation.copy(node.mesh.rotation);
//         }
//     });
// }

function checkStabilization() {
    const allNodesStable = nodes.every((node) => node.velocity.length() < minVelocityThreshold);
    if (allNodesStable) {
        stabilizedFrames += 1;
    } else {
        stabilizedFrames = 0;
    }
    if (stabilizedFrames >= stabilizationFrames) {
        graphStabilized = true;
        console.log("Graph stabilized.");
    }
}

const loadGenerateGallery = () => {
    sceneMode.value = 'generateGallery'
    hideGroup([demoGalleryGroup, userGalleryGroup, analyseGroup, generateGroup, generateGalleryGroup])
    //showGroup([generateGalleryGroup])

    //here we also need to check whether we are currently looking at the demogallery or the usergallery
    //and if it is the demogallery we need to search for the active image in the demoGalleryGroup
    if (activeImage.value) {
        userGalleryGroup.children.forEach(child => {
            if (child.name === activeImage.value?.userData.image.file_name) {
                child.visible = true;
            }
        })
        // generateGalleryGroup.children.forEach(child => {
        //     if (child.name === activeImage.value?.userData.image.file_name) {
        //         child.visible = true;
        //     }
        // })
    }

    //moveCameraToPosition(new THREE.Vector3(0, 0, 0))
    console.log('we are in generateGallery mode');
    //am not sure if i need this next function, because as such we are not calling it
    //however, we need the showing of the associatedObjects which is now located inside this function
    //am not using the variable associatedObjects for now - so am going to comment it out
    // const associatedObjects = generateGalleryGroup.children.filter((child) => {
    //     if (child.userData.name === activeImage.value?.userData.image.file_name) {
    //         //make objects visible that are associated with the activeImage
    //         child.visible = true;
    //         if (child.userData.edgesMesh) {
    //             child.userData.edgesMesh.visible = true;
    //         }
    //         return true;
    //     }
    //     return false;
    // });

    //the same functionality of filtering and toggling visibility - without assigning to a variable
    generateGalleryGroup.children.forEach((child) => {
        if (child.userData.name === activeImage.value?.userData.image.file_name) {
            // Make objects visible that are associated with the activeImage
            child.visible = true;
            if (child.userData.edgesMesh) {
                child.userData.edgesMesh.visible = true;
            }
        }
    });


    //console.log('associatedObjects:', associatedObjects);
    //visualizeForceGraph(activeImage.value, associatedObjects, scene);
    if (activeImage.value) {
        graphStabilized = false;
        emptyGroup(linesForceGraphGroup);
        setupForceGraphFromGallery(activeImage.value, generateGalleryGroup);
    } else {
        console.error("No active image available.");
    }
}


</script>
<style scoped></style>