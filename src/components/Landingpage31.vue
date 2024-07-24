<template>
    <div ref="canvasContainer" class="h-screen w-screen relative">
        <div class="absolute inset-0 flex justify-center items-center" style="pointer-events: none;">
            <button style="pointer-events: auto;" class="h-10 px-3 w-fit bg-slate-200 rounded-lg"
                @click="leave">START</button>
        </div>
        <div>
            <button style="pointer-events: auto;"
                class="absolute top-10 left-10 h-10 px-3 w-fit bg-slate-200 rounded-lg"
                @click="startAnimation">cloud</button>
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
// let renderer: THREE.WebGLRenderer | undefined; // Define renderer at the top level
let controls: OrbitControls | undefined; // Define controls at the top level

const densityScalingFactor = 80000; // Adjust this value as needed

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer({ antialias: true });
//const renderer = new THREE.WebGLRenderer({ antialias: false }); 
renderer.setSize(window.innerWidth, window.innerHeight);
// add antialiasing



//empty array for batched mesh ids
const batchedMeshIds: number[] = [];
let batchedMesh = new THREE.BatchedMesh(0, 0, 0);
const isAnimating = ref(true); // Flag to control the animation loop

const initThree = () => {

    const loader = new THREE.TextureLoader();
    loader.load('/texture_atlas.jpg', (texture) => {
        console.log('Texture loaded:', texture);

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
                console.log('CSV data:', results.data);

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

                    // Create the material and mesh
                    //const material = new THREE.MeshBasicMaterial({ map: texture, side: THREE.DoubleSide });
                    // const plane = new THREE.Mesh(geometry, material);
                    // plane.position.set(tsneX, density3 * densityScalingFactor, tsneY); // Use tsne_x for x, scaled density_3 for y, and tsne_y for z

                    // Add the mesh to the scene
                    // scene.add(plane);
                });

                // Position the camera
                camera.position.z = 10;

                // Start the animation loop
                //animate();
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
                const validPositions = results.data.filter((data: any) => data.tsne_x && data.tsne_y && data.density_3);
                resolve(validPositions);
            },
            error: (error) => reject(error)
        });
    });

    return parsedPositions;
}

// usage of updatePositionsFromCSV
async function updatePositionsFromCSV(csvFile: string) {
    //const csvFilePath = '/tsne_coordinates_pca_r42_2d_V2.csv';
    let newPositions: any[] = []; // Explicitly type newPositions as an array of any type and initialize it as an empty array
    try {
        newPositions = await loadPositionsFromCSV(csvFile) as any[];
        //console.log('new positions: ', newPositions);
    } catch (error) {
        console.error('Error loading positions from CSV:', error);
    }
    //console.log('new positions: ', newPositions);
    // now animate map the new positions to the mesh
    // if (!Array.isArray(batchedMeshIds) || batchedMeshIds.length === 0) {
    //     console.warn('batchedMeshIds is not a valid array or is empty');
    //     return; // Stop the function if batchedMeshIds is not valid
    // }
    if (batchedMeshIds.length === 0) {
        console.warn('batchedMeshIds is empty');
        return;
    }
    batchedMeshIds.forEach((geoId, i) => {
        //console.log('trying to do stuff');
        try {
            const matrix4 = batchedMesh.getMatrixAt(geoId, new THREE.Matrix4());
            // Rotate the mesh via matrix
            const ownPosition = new THREE.Vector3();
            matrix4.decompose(ownPosition, new THREE.Quaternion(), new THREE.Vector3());
            // Attempt to modify the matrix. If this fails, the catch block will handle the error.
            matrix4.lookAt(ownPosition, camera.position, new THREE.Vector3(0, 1, 0));
            batchedMesh.setMatrixAt(geoId, matrix4);

        } catch (error) {
            console.error(`Error transforming object with geoId ${geoId}:`, error);
            // Optionally, handle the error further or continue to the next iteration.
        }

        //// only tween approach
        const matrix4 = batchedMesh.getMatrixAt(geoId, new THREE.Matrix4());
        //const initialPosition = initialPositions[i];
        const initialPosition = new THREE.Vector3();
        matrix4.decompose(initialPosition, new THREE.Quaternion(), new THREE.Vector3());
        const tsneX = parseFloat(newPositions[i].tsne_x);
        //console.log('tsneX:', tsneX);
        const tsneY = parseFloat(newPositions[i].tsne_y);
        const density3 = parseFloat(newPositions[i].density_3);
        const targetPosition = { x: tsneX, y: density3 * densityScalingFactor, z: tsneY };

        const matrix4a = new THREE.Matrix4();
        matrix4a.makeTranslation(tsneX, density3 * densityScalingFactor, tsneY);
        //                     batchedMesh.setMatrixAt(geoId, matrix4);

        //const targetPosition = newPositions;
        const dummy = new THREE.Object3D();
        new TWEEN.Tween(initialPosition)
            .to(targetPosition, 2000 + Math.random() * 2000) // Randomize duration for each instance
            .easing(TWEEN.Easing.Quadratic.Out)
            .onUpdate(() => {
                dummy.position.set(initialPosition.x, initialPosition.y, initialPosition.z);
                dummy.updateMatrix();
                batchedMesh.setMatrixAt(i, dummy.matrix);
                //batchedMesh.instanceMatrix.needsUpdate = true;
            })
            .start();



    });

}

// Function to be called when 'cloud' button is clicked
const startAnimation = () => {
    updatePositionsFromCSV('/tsne_coordinates_pca_r42_2d_V2.csv')
};
//tsne_coordinates_pca_r0_3d.csv //this is for the cloud button



// for (let i = 0; i < count; i++) {
//             const initialPosition = initialPositions[i];
//             const targetPosition = targetPositions[i];
//             const dummy = new THREE.Object3D();

//             new TWEEN.Tween(initialPosition)
//                 .to(targetPosition, 8000 + Math.random() * 2000) // Randomize duration for each instance
//                 .easing(TWEEN.Easing.Quadratic.Out)
//                 .onUpdate(() => {
//                     dummy.position.set(initialPosition.x, initialPosition.y, initialPosition.z);
//                     dummy.updateMatrix();
//                     batchedMesh.setMatrixAt(i, dummy.matrix);
//                     batchedMesh.instanceMatrix.needsUpdate = true;
//                 })
//                 .start();
//         }

//
// batchedMeshIds.push(geoId);
//                     const matrix4 = new THREE.Matrix4();
//                     matrix4.makeTranslation(tsneX, density3 * densityScalingFactor, tsneY);
//                     batchedMesh.setMatrixAt(geoId, matrix4);

//exampleUsage();
//updatePositionsFromCSV('/tsne_coordinates_pca_r42_2d_V2.csv')

// async function updatePositionsFromCSV(csvFile: string) {
//     const response = await fetch(csvFile);
//     const csvText = await response.text();
//     Papa.parse(csvText, {
//         complete: (results) => {
//             results.data.forEach((row, index) => {
//                 const [x, y, z] = (row as number[]).map(Number);
//                 if (batchedMeshIds.includes(index)) {
//                     const newPosition = { x, y, z };
//                     animateMeshToPosition(batchedMesh.getObjectByIndex(index), newPosition);
//                     //batchedMeshIds.push(geoId);
//                     const matrix4 = new THREE.Matrix4();
//                     matrix4.makeTranslation(tsneX, density3 * densityScalingFactor, tsneY);
//                     batchedMesh.setMatrixAt(geoId, matrix4);
//                 }
//             });
//         }
//     });
// }

// async function loadPositionsFromCSV(csvFilePath: string) {
//     // Parse the CSV file
//     // For each row in the CSV, extract the new position and store it in an array or object
//     Papa.parse(csvFilePath, {
//         download: true,
//         header: true,
//         complete: (results: Papa.ParseResult<{}>) => {
//             console.log('CSV data new:', results.data);
//                          results.data.forEach((data: any) => {
//                             // Check if the row is valid
//                             if (!data.tsne_x || !data.tsne_y || !data.density_3) {
//                                 console.warn(`Skipping invalid row: ${JSON.stringify(data)}`);
//                                 return;
//                             }
//                         }
//                          )
//                     }
//         }); // end papa parse



//     const parsedPositions: any[] = results.data; // Initialize parsedPositions
//     return parsedPositions;
// }

// const newPosition = loadPositionsFromCSV('/tsne_coordinates_pca_r42_2d_V2.csv')
// //const newPosition = loadPositionsFromCSV('/tsne_coordinates_pca_r0_2d_V2_res64_uv.csv')

// console.log('new positions: ', newPosition);

// function animateObjectsToNewPositions(objects, newPositions) {
//     const tween = new TWEEN.Tween(mesh.position)
//         .to(newPosition, 2000) // 2000 ms = 2 seconds
//         .easing(TWEEN.Easing.Quadratic.Out) // Use any easing function you like
//         .onUpdate(() => {
//             // This will be called on every frame during the tween
//             // You can update something else here if needed
//         })
//         .start(); // Start the tween immediately
// }



// // Function to be called when 'cloud' button is clicked
// const startAnimation = () => {
//   updatePositionsFromCSV('/tsne_coordinates_pca_r42_2d.csv');
// };

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
        // batchedMeshIds.forEach((geoId) => {
        //     const geometry = batchedMesh.geometry(geoId);
        //     if (geometry) {
        //         geometry.dispose();
        //     }
        // });
        console.log(batchedMesh.geometry);
        batchedMeshIds.splice(0, batchedMeshIds.length);
        console.log(batchedMeshIds.length);
        batchedMesh.dispose();
        //batchedMesh.geometry.dispose();
        //batchedMesh.material.dispose();
        scene.remove(batchedMesh);
        //console.log(batchedMesh.geometry);
    }



    window.removeEventListener("resize", setSize);

});
</script>
<style scoped></style>