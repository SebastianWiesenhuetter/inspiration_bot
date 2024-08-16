<template>
    <div class="h-screen w-screen">
        <div ref='threejsMap' @click="onClick" @mousemove="onDrag" @mousedown="isMouseDown = true"
            @touchstart="isMouseDown = true" @mouseup="mouseUp" @touchend="mouseUp">
            <div class="absolute top-2 h-auto w-full flex flex-row justify-between space-x-1">
                <button class=" h-10 w-full bg-slate-200 rounded-lg"
                    @click.stop="sceneMode = 'gallery'">gallery</button>
                <button class=" h-10 w-full bg-slate-200 rounded-lg"
                    @click.stop="sceneMode = 'analyse'">analyse</button>
                <button class=" h-10 w-full bg-slate-200 rounded-lg"
                    @click.stop="sceneMode = 'generate'">generate</button>
            </div>
            <div v-if="sceneMode == 'analyse'"
                class="absolute top-1/3 right-80 h-1/2 w-fit flex flex-col justify-start items-center space-y-1 pointer-events-none">
                <button
                    class="h-10 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 active:bg-slate-300 pointer-events-auto"
                    :class="houghLinesVisible ? 'bg-slate-400' : ''" @click.stop="showHoughLines">semantic
                    lines</button>
                <button
                    class="h-10 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 active:bg-slate-300 pointer-events-auto "
                    :class="segmentationVisible ? 'bg-slate-400' : ''"
                    @click.stop="showSegmentationGeometry">segmentation</button>
                <button
                    class="h-10 w-fit px-4 bg-slate-200 rounded-lg hover:bg-slate-400 active:bg-slate-300 pointer-events-auto "
                    :class="paintingSequenceVisible ? 'bg-slate-400' : ''"
                    @click.stop="showPaintingSequence">painting sequence</button>
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
//import { randFloat } from 'three/src/math/MathUtils.js';



// Create a scene
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x707070);
// // Add a directional light
// const directionalLight = new THREE.DirectionalLight(0xffffff, 1.4);
// directionalLight.position.set(-2, 5, 4);
// scene.add(directionalLight);

// // Add an ambient light
// const ambientLight = new THREE.AmbientLight(0x404040, 30); // Soft white light
// scene.add(ambientLight);



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

const houghLinesVisible = ref(false)
const segmentationVisible = ref(false)
const paintingSequenceVisible = ref(false)

//STATS for FPS monitoring
const stats = new Stats();
stats.showPanel(0); // 0: fps, 1: ms, 2: mb, 3+: custom
document.body.appendChild(stats.dom);

onMounted(() => {
    stats.begin();
    threejsMap.value?.appendChild(domElement);
    loadGallery();
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
const sceneMode = ref<'gallery' | 'analyse' | 'generate'>('gallery')
//const sceneMode = ref<'gallery' | 'analyse' | 'generate'>('analyse')


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

const galleryGroup: THREE.Group = new THREE.Group();
galleryGroup.name = 'galleryGroup'
scene.add(galleryGroup);
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
type ImageData = {
    file_name: string;
    resolution: {
        width: number;
        height: number;
    };
}
const isGalleryLoaded = ref(false)
const loadGallery = async () => {

    if (isGalleryLoaded.value) {
        showGroup([galleryGroup])
        tweenToPosition(lastCameraPosition)
        tweenToCameraTarget(controls.target, lastCameraTarget)
        hideGroup([analyseGroup, generateGroup])
        return
    }
    // get imageJson from puplic folder
    //const offset = ref(0) //NOT REQUIRED??
    await fetch('/image_resolutions100.json')
        .then(response => response.json())
        .then(data => {
            data.forEach((image: ImageData) => {
                const geometry = new THREE.PlaneGeometry(1, 1);
                const texture = new THREE.TextureLoader().load(`/sample_images_10k_orig01/${image.file_name}`);
                const material = new THREE.MeshBasicMaterial({ map: texture, side: THREE.DoubleSide });
                //const material = new THREE.MeshLambertMaterial({ map: texture, side: THREE.DoubleSide });
                const plane = new THREE.Mesh(geometry, material);
                plane.scale.set(image.resolution.width / image.resolution.height, 1, 1);
                plane.visible = true
                //plane.translateX(offset.value) //NOT REQUIRED??
                plane.userData = { image }
                plane.userData.isImage = true;
                plane.name = image.file_name;
                galleryGroup.add(plane);
            });
        });
    arrangeIn2DGrid(10, 0.1, galleryGroup.children)
    isGalleryLoaded.value = true
    sceneMode.value = 'gallery'
    hideGroup([analyseGroup, generateGroup])
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
const loadAnalyse = () => {
    lastCameraPosition.copy(camera.position) //will this work also - if we directly start in analyse mode? i think yes, since we declare it as a global variable
    lastCameraTarget.copy(controls.target)
    hideGroup([galleryGroup, generateGroup])
    houghLinesVisible.value = false;
    segmentationVisible.value = false;
    paintingSequenceVisible.value = false;
    //do we need this next loop?	
    analyseGroup.children.forEach(child => {
        //analyseGroup.remove(child)
        // dispose everything from child 
        child.visible = false;
    })
    if (activeImage.value) {
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
        }

        //for each analyseGroup#

        //analyseGroup.children[0].visible = true
        moveCameraToPosition(activeImage.value.position)
    }
    // this part does not work yet - we have to create a random image loading routine from the whole of the dataset, not only from the gallery
    // probably this part should be removed later in it current form
    // else {
    //     activeImage.value = galleryGroup.children[randFloat(0, galleryGroup.children.length - 1)]
    //     console.log('am in analyse mode: ',activeImage.value);
    // }
}

const lastCameraPosition = new THREE.Vector3(0,0,5) //set initial camera position has to be reconsidered - but still better than leaving it empty, otherwise zooms into nothing
const lastCameraTarget = new THREE.Vector3(0,0,0)
const moveCameraToPosition = (position: THREE.Vector3) => {
    //this needs to be moved out of this function - but then called every time - we want to remember the last camera position
    // for now, i already moved it on top of the loadAnalyse function
    // lastCameraPosition.copy(camera.position)
    // lastCameraTarget.copy(controls.target)
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
    hideGroup([galleryGroup, analyseGroup])
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
    const intersects = raycaster.intersectObjects(galleryGroup.children);
    if (intersects.length > 0 && !isDragging.value) {
        const object = intersects[0].object;
        if (object.userData.isImage && object.visible) {
            activeImage.value = object
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

const showHoughLines = async () => {
    let houghLineGroup = scene.getObjectByName('houghLineGroup_' + activeImage.value?.userData.image.file_name);
    if (houghLineGroup) {
        houghLineGroup.visible = !houghLineGroup.visible
        houghLinesVisible.value = houghLineGroup.visible
        return;
    }
    else {
        houghLineGroup = new THREE.Group();
        houghLineGroup.name = 'houghLineGroup_' + activeImage.value?.userData.image.file_name;
        analyseGroup.add(houghLineGroup);
        houghLinesVisible.value = true
    }
    const image = activeImage.value?.userData.image;
    const imagePosition = activeImage.value?.position; // Get the position of the image plane
    // Get image dimensions
    const imageHeight = image.resolution.height;

    // fetch hough lines from json file 
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
        houghLineGroup.add(lineObject);
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
        if(color){
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
        const paintingSequenceLayerGroup = createSegmentationShape(contourLayer, imagePosition, image,color)
        paintingSequenceLayerGroup.position.z += 0.01 + index * offset;
        paintingSequenceGroup.add(paintingSequenceLayerGroup);

    });

}


</script>
<style scoped></style>