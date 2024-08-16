<template>
    <div ref="canvasContainer" class="h-screen w-screen relative">
        <div class="absolute inset-0 flex justify-center items-end mb-10" style="pointer-events: none;">
            <button style="pointer-events: auto;" class="h-10 px-3 w-fit bg-slate-200 rounded-lg"
                @click="leave">START</button>
        </div>
        <div v-if="isCsvLoaded">
            <button style="pointer-events: auto;" class="absolute bottom-5 right-5 " @click="startAnimation('Cloud')">
                <img src="/cloud100px_8.png" alt="landscape" border="0" width="100%" height="100%" /></button>
            <button style="pointer-events: auto;" class="absolute bottom-24 right-5 "
                @click="startAnimation('Landscape')">
                <img src="/landscape_button07.png" alt="cloud" border="0" width="100%" height="100%" /></button>
        </div>
        <div id="threejs-container"></div>
    </div>
</template>

<script setup lang="ts">
import { onMounted, onUnmounted, ref, defineEmits } from 'vue';
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import Papa from 'papaparse';
import Stats from 'three/examples/jsm/libs/stats.module.js';
import * as TWEEN from '@tweenjs/tween.js';

const emit = defineEmits(['leave']);
const leave = () => {
    isAnimating.value = false;
    emit('leave');
};

const canvasContainer = ref<HTMLElement | null>(null);

const isCsvLoaded = ref(false);
let controls: OrbitControls | undefined; // Define controls at the top level

let densityScalingFactor = 80000; // Adjust this value as needed

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer({ antialias: true });
//const renderer = new THREE.WebGLRenderer({ antialias: false }); 
renderer.outputColorSpace = THREE.LinearSRGBColorSpace;
renderer.setSize(window.innerWidth, window.innerHeight);

//empty array for batched mesh ids
const batchedMeshIds: number[] = [];
let batchedMesh = new THREE.BatchedMesh(0, 0, 0);
const isAnimating = ref(true); // Flag to control the animation loop

const initThree = () => {

    const loader = new THREE.TextureLoader();
    loader.load('/texture_atlas.jpg', (texture) => {
        const material = new THREE.MeshBasicMaterial({ map: texture, side: THREE.DoubleSide });
        const numberOfMeshes = 10000;
        batchedMesh = new THREE.BatchedMesh(numberOfMeshes, numberOfMeshes * 4, numberOfMeshes * 10, material);
        scene.add(batchedMesh);
        loadCSVData(batchedMesh);
    }, undefined, (err) => {
        console.error('An error occurred loading the texture:', err);
    });


    const loadCSVData = (batchedMesh: THREE.BatchedMesh) => {
        Papa.parse('/tsne_coordinates_pca_r0_2d_V2_res64_uv.csv', {
            download: true,
            header: true,
            complete: (results: Papa.ParseResult<{}>) => {
                //console.log('CSV data:', results.data);
                results.data.forEach((data: any) => {
                    // Check if the row is valid
                    if (!data.image_width || !data.image_height || !data.u_min || !data.v_min || !data.u_max || !data.v_max || !data.density_3) {
                        console.warn(`Skipping invalid row: ${JSON.stringify(data)}`);
                        return;
                    }

                    const imgWidth = parseFloat(data.image_width);
                    const imgHeight = parseFloat(data.image_height);
                    const tsneX = parseFloat(data.tsne_x);
                    const tsneY = parseFloat(data.tsne_y);
                    const density3 = parseFloat(data.density_3);

                    // Define a uniform height
                    const uniformHeight = 1;
                    const aspectRatio = imgWidth / imgHeight;
                    const calculatedWidth = uniformHeight * aspectRatio;
                    // console.log(`Creating geometry for ${data.image_filename}: width=${calculatedWidth}, height=${uniformHeight}`);

                    const geometry = new THREE.PlaneGeometry(calculatedWidth, uniformHeight);

                    // Adjust UV mapping to the correct coordinates
                    const uMin = parseFloat(data.u_min);
                    const vMin = parseFloat(data.v_min);
                    const uMax = parseFloat(data.u_max);
                    const vMax = parseFloat(data.v_max);

                    const uv = geometry.attributes.uv;
                    uv.array[0] = uMin; // Bottom left
                    uv.array[1] = vMax;
                    uv.array[2] = uMax; // Bottom right
                    uv.array[3] = vMax;
                    uv.array[4] = uMin; // Top left
                    uv.array[5] = vMin;
                    uv.array[6] = uMax; // Top right
                    uv.array[7] = vMin;
                    uv.needsUpdate = true;

                    const geoId = batchedMesh.addGeometry(geometry);
                    // console.log(geoId)
                    batchedMeshIds.push(geoId);
                    const matrix4 = new THREE.Matrix4();
                    matrix4.makeTranslation(tsneX, density3 * densityScalingFactor, tsneY);
                    batchedMesh.setMatrixAt(geoId, matrix4);
                });

                // Position the camera
                //camera.position.z = 10;
                camera.position.set(-23.2, 36.6, -44.4);
                controls!.target.set(-7.4, -7.6, -4.3);
                //controls: 

            },
            error: (err) => {
                console.error('Error parsing CSV:', err);
            }
        });
    };

    controls = new OrbitControls(camera, renderer!.domElement); // Initialize controls
    controls.enableDamping = true;
    controls.dampingFactor = 0.05;
    controls.autoRotate = false;
    controls.autoRotateSpeed = 0.5;

    const animate = () => {
        stats.update();
        if (!isAnimating.value) {
            console.log('Stopping animation loop');
            return;
        } // Stop the animation loop if the flag is false
        requestAnimationFrame(animate);
        TWEEN.update();


        if (!Array.isArray(batchedMeshIds) || batchedMeshIds.length === 0) {
            console.warn('batchedMeshIds is not a valid array or is empty');
            return; // Stop the function if batchedMeshIds is not valid
        }
        batchedMeshIds.forEach((geoId) => {
            try {
                const matrix4 = batchedMesh.getMatrixAt(geoId, new THREE.Matrix4());
                // Rotate the mesh via matrix
                const ownPosition = new THREE.Vector3();
                matrix4.decompose(ownPosition, new THREE.Quaternion(), new THREE.Vector3());
                // Attempt to modify the matrix. If this fails, the catch block will handle the error.
                matrix4.lookAt(ownPosition, camera.position, new THREE.Vector3(0, 1, 0));
                batchedMesh.setMatrixAt(geoId, matrix4);
                if (renderer.info.render.triangles == 0) {
                    // Code to execute if the condition is true
                    //console.log('own position:', matrix4);
                }
                //console.log('Rendered triangles:', renderer.info.render.triangles);
            } catch (error) {
                console.error(`Error transforming object with geoId ${geoId}:`, error);
                // Optionally, handle the error further or continue to the next iteration.
            }
        });
        controls!.update(); // Update controls in the animation loop
        renderer!.render(scene, camera);
    };

    animate();
    const threejsContainer = document.getElementById('threejs-container');
    if (threejsContainer) {
        threejsContainer.appendChild(renderer.domElement);
    }
};


const setSize = () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();

    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setPixelRatio(window.devicePixelRatio);
};

// STATS for FPS monitoring
const stats = new Stats();
stats.showPanel(0); // 0: fps, 1: ms, 2: mb, 3+: custom
document.body.appendChild(stats.dom);



/////////////// TWEEN STUFF ///////////////////////

async function loadPositionsFromCSV(csvFilePath: string) {
    // Wrap Papa.parse in a promise to handle the asynchronous nature
    const parsedPositions = await new Promise((resolve, reject) => {
        Papa.parse(csvFilePath, {
            download: true,
            header: true,
            complete: (results) => {
                // Filter out invalid rows directly in the promise
                const validPositions = results.data.filter((data: any) => data.tsne_x && data.tsne_y && (data.density_3 || data.tsne_z));
                resolve(validPositions);
            },
            error: (error) => reject(error)
        });
    });

    return parsedPositions;
}

//lets load all the 4 csv files directly at the start
let positionsCloud1: any[] = [];
let positionsCloud2: any[] = [];
let positionsLandscape1: any[] = [];
let positionsLandscape2: any[] = [];
const loadAllPositions = async () => {
    positionsCloud1 = await loadPositionsFromCSV('/tsne_coordinates_pca_r0_3d.csv') as any[];
    positionsCloud2 = await loadPositionsFromCSV('/tsne_coordinates_pca_r42_3d.csv') as any[];
    positionsLandscape1 = await loadPositionsFromCSV('/tsne_coordinates_pca_r0_2d_V2_res64_uv.csv') as any[];
    positionsLandscape2 = await loadPositionsFromCSV('/tsne_coordinates_pca_r42_2d_V2.csv') as any[];
    //what are the different versions of the csv files?
    //now we have used the versions with pca initialisation
    //we could compare them to the ones without pca initialisation
    //tsne_coordinates_noi_r0_2d_V2.csv
    //tsne_coordinates_noi_r42_2d_V2.csv
    //tsne_coordinates_noi_r0_3d.csv
    //tsne_coordinates_noi_r42_3d.csv

};

loadAllPositions().then(() => {
    isCsvLoaded.value = true;
});


// usage of updatePositionsFromCSV
// async function updatePositionsFromCSV(csvFile: string) {
//     //const csvFilePath = '/tsne_coordinates_pca_r42_2d_V2.csv';
//     let newPositions: any[] = []; // Explicitly type newPositions as an array of any type and initialize it as an empty array
//     try {
//         newPositions = await loadPositionsFromCSV(csvFile) as any[];
//     } catch (error) {
//         console.error('Error loading positions from CSV:', error);
//     }
//     //console.log('new positions: ', newPositions);
//     if (batchedMeshIds.length === 0) {
//         console.warn('batchedMeshIds is empty');
//         return;
//     }
//     batchedMeshIds.forEach((geoId, i) => {

//         //// only tween approach
//         const matrix4 = batchedMesh.getMatrixAt(geoId, new THREE.Matrix4());
//         //const initialPosition = initialPositions[i];
//         const initialPosition = new THREE.Vector3();
//         matrix4.decompose(initialPosition, new THREE.Quaternion(), new THREE.Vector3());
//         const tsneX = parseFloat(newPositions[i].tsne_x);
//         //console.log('tsneX:', tsneX);
//         const tsneY = parseFloat(newPositions[i].tsne_y);
//         let yValue = 0;
//         if (newPositions[i].density_3 != undefined) {
//             yValue = parseFloat(newPositions[i].density_3);
//             densityScalingFactor = 80000;
//         }
//         else if (newPositions[i].tsne_z != undefined) {
//             yValue = parseFloat(newPositions[i].tsne_z);
//             densityScalingFactor = 0.5;
//         }

//         const targetPosition = { x: tsneX, y: yValue * densityScalingFactor, z: tsneY };

//         const matrix4a = new THREE.Matrix4();
//         matrix4a.makeTranslation(tsneX, yValue * densityScalingFactor, tsneY);

//         /// TODO - probably remove dummy object
//         const dummy = new THREE.Object3D();

//         new TWEEN.Tween(initialPosition)
//             .to(targetPosition, 2000 + Math.random() * 2000) // Randomize duration for each instance
//             .easing(TWEEN.Easing.Quadratic.Out)
//             .onUpdate(() => {
//                 dummy.position.set(initialPosition.x, initialPosition.y, initialPosition.z);
//                 dummy.updateMatrix();
//                 batchedMesh.setMatrixAt(i, dummy.matrix);
//                 //batchedMesh.instanceMatrix.needsUpdate = true;
//             })
//             .start();

//     });

// }
async function updatePositionsFromCSV1(newPositions: any[]) {

    batchedMeshIds.forEach((geoId, i) => {

        //// only tween approach
        const matrix4 = batchedMesh.getMatrixAt(geoId, new THREE.Matrix4());
        //const initialPosition = initialPositions[i];
        const initialPosition = new THREE.Vector3();
        matrix4.decompose(initialPosition, new THREE.Quaternion(), new THREE.Vector3());
        const tsneX = parseFloat(newPositions[i].tsne_x);
        //console.log('tsneX:', tsneX);
        const tsneY = parseFloat(newPositions[i].tsne_y);
        let yValue = 0;
        if (newPositions[i].density_3 != undefined) {
            yValue = parseFloat(newPositions[i].density_3);
            densityScalingFactor = 80000;
        }
        else if (newPositions[i].tsne_z != undefined) {
            yValue = parseFloat(newPositions[i].tsne_z);
            densityScalingFactor = 0.5;
        }

        const targetPosition = { x: tsneX, y: yValue * densityScalingFactor, z: tsneY };

        const matrix4a = new THREE.Matrix4();
        matrix4a.makeTranslation(tsneX, yValue * densityScalingFactor, tsneY);

        /// TODO - probably remove dummy object
        const dummy = new THREE.Object3D();

        new TWEEN.Tween(initialPosition)
            .to(targetPosition, 2000 + Math.random() * 2000) // Randomize duration for each instance
            //.easing(TWEEN.Easing.Quadratic.Out)
            //.easing(TWEEN.Easing.Quartic.Out)
            .easing(TWEEN.Easing.Elastic.Out)
            .onUpdate(() => {
                dummy.position.set(initialPosition.x, initialPosition.y, initialPosition.z);
                dummy.updateMatrix();
                batchedMesh.setMatrixAt(i, dummy.matrix);
                //batchedMesh.instanceMatrix.needsUpdate = true;
            })
            //         .onComplete(function() {
            //     scene.remove(dummy) // Dispose of the dummy object after the tween is complete
            //     // You can also set a flag here if you need to check the completion status later
            // })
            .start();

    });

}

// Function to be called when 'cloud' button is clicked
const cloudPositionState = ref(0);
const landscapePositionState = ref(0);
const startAnimation = (type: 'Landscape' | 'Cloud') => {
    TWEEN.removeAll();
    if (type === 'Cloud') {
        if (cloudPositionState.value === 0) {
            updatePositionsFromCSV1(positionsCloud2);
            // updatePositionsFromCSV('/tsne_coordinates_pca_r42_3d.csv'); 
            ////tsne_coordinates_pca_r42_3d.csv
            cloudPositionState.value = 1;
        } else {
            updatePositionsFromCSV1(positionsCloud1);
            // updatePositionsFromCSV('/tsne_coordinates_pca_r0_3d.csv'); 

            cloudPositionState.value = 0;
        }
    } else {
        if (landscapePositionState.value === 0) {
            updatePositionsFromCSV1(positionsLandscape2);
            // updatePositionsFromCSV('/tsne_coordinates_pca_r42_2d_V2.csv');
            landscapePositionState.value = 1;
        } else {
            updatePositionsFromCSV1(positionsLandscape1);
            // updatePositionsFromCSV('/tsne_coordinates_pca_r0_2d_V2_res64_uv.csv');
            landscapePositionState.value = 0;
        }
    }
};
//tsne_coordinates_pca_r0_3d.csv //this is for the cloud button

//// END TWEEN STUFF ///////////////////////


onMounted(() => {
    stats.begin();
    initThree();
    window.addEventListener("resize", setSize);
});

onUnmounted(() => {
    if (renderer && renderer.domElement.parentNode) {
        renderer.domElement.parentNode.removeChild(renderer.domElement);
    }

    // Dispose of renderer
    if (renderer) {
        renderer.dispose();
        if (renderer.domElement.parentNode) {
            renderer.domElement.parentNode.removeChild(renderer.domElement);
        }
    }
    // Dispose of batchedMesh and related resources
    if (batchedMesh) {
        //console.log(batchedMesh.geometry);
        batchedMeshIds.splice(0, batchedMeshIds.length);
        console.log(batchedMeshIds.length);
        batchedMesh.dispose();
        //batchedMesh.geometry.dispose();
        //batchedMesh.material.dispose();
        scene.remove(batchedMesh);
        //console.log(batchedMesh.geometry);
    }


    // Dispose of tween resources
    TWEEN.removeAll();

    window.removeEventListener("resize", setSize);

});
</script>
<style scoped></style>