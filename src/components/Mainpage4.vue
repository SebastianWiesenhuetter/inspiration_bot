<template>
    <div class="h-screen w-screen">
        <div ref='threejsMap' @click="onClick" @mousemove="onDrag" @mousedown="isMouseDown = true"
            @touchstart="isMouseDown = true" @mouseup="mouseUp" @touchend="mouseUp">
            <div class="absolute top-2 w-full flex flex-row justify-between space-x-1">
                <button class=" h-10 w-full bg-slate-200 rounded-lg"
                    @click.stop="sceneMode = 'gallery'">gallery</button>
                <button class=" h-10 w-full bg-slate-200 rounded-lg"
                    @click.stop="sceneMode = 'analyse'">analyse</button>
                <button class=" h-10 w-full bg-slate-200 rounded-lg"
                    @click.stop="sceneMode = 'generate'">generate</button>
            </div>
            <div v-if="sceneMode == 'analyse'">
                <button class="absolute top-1/3 right-80 h-10 w-fit px-4 bg-slate-200 rounded-lg"
                    @click.stop="showHoughLines">deep hough</button>
            </div>


        </div>
        <div id="fpsCounter"
     style="position: absolute; top: 48px; left: 10px; color: white; background: rgba(0, 0, 0, 0.5); padding: 5px; border-radius: 5px;">
    FPS: {{ fps }}
</div>
    </div>
</template>
<script setup lang="ts">
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import * as TWEEN from '@tweenjs/tween.js';
import { onMounted, ref, watch } from 'vue';
import { Line2 } from 'three/examples/jsm/lines/Line2';
import { LineGeometry } from 'three/examples/jsm/lines/LineGeometry';
import { LineMaterial } from 'three/examples/jsm/lines/LineMaterial';

//FPS measuring 2be deleted later
const fps = ref(0);
let lastTime = performance.now();
const updateFPS = () => {
    const now = performance.now();
    const delta = now - lastTime;
    fps.value = parseFloat((1000 / delta).toFixed(2));
    lastTime = now;
};

// Create a scene
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x525252);
// Add a directional light
const directionalLight = new THREE.DirectionalLight(0xffffff, 1.4);
directionalLight.position.set(-2, 5, 4);
scene.add(directionalLight);

// Add an ambient light
const ambientLight = new THREE.AmbientLight(0x404040, 10); // Soft white light
scene.add(ambientLight);

// Create a camera
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
camera.position.z = 5;

// Create a renderer
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
// renderer.toneMapping = THREE.ACESFilmicToneMapping

// Create controls
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;

// Create raycaster
const raycaster = new THREE.Raycaster();

const threejsMap = ref<Node>()

const domElement = renderer.domElement;

onMounted(() => {
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
    requestAnimationFrame(animate);

    //FPS measuring 2be deleted later
    updateFPS();


    renderer.render(scene, camera);
    controls.update();
    TWEEN.update();
}
const sceneMode = ref<'gallery' | 'analyse' | 'generate'>('gallery')

watch(() => sceneMode.value, (newVal, oldVal) => {
    if (newVal === oldVal) return
    if (newVal === 'gallery') {
        loadGallery()
    } else if (newVal === 'analyse') {
        loadAnalyse()
    } else if (newVal === 'generate') {
        loadExtra()
    }
})

const galleryGroup: THREE.Group = new THREE.Group();
galleryGroup.name = 'galleryGroup'
scene.add(galleryGroup);
const analyseGroup: THREE.Group = new THREE.Group();
analyseGroup.name = 'analyseGroup'
scene.add(analyseGroup);
const extraGroup: THREE.Group = new THREE.Group();
extraGroup.name = 'extraGroup'
scene.add(extraGroup);


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
        hideGroup([analyseGroup, extraGroup])
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
                const plane = new THREE.Mesh(geometry, material);
                plane.scale.set(image.resolution.width / image.resolution.height, 1, 1);
                plane.visible = true
                //plane.translateX(offset.value) //NOT REQUIRED??
                plane.userData = { image }
                plane.userData.isImage = true
                galleryGroup.add(plane);
            });
        });
    arrangeIn2DGrid(5, 20, 0.1, galleryGroup.children)
    isGalleryLoaded.value = true
    sceneMode.value = 'gallery'
    hideGroup([analyseGroup, extraGroup])
}
const arrangeIn2DGrid = (rows: number, cols: number, margin: number, group: THREE.Object3D[]) => {
    let i = 0;
    for (let row = 0; row < rows; row++) {
        for (let col = 0; col < cols; col++) {
            if (i >= group.length) return
            const obj = group[i];
            obj.position.set(col * (1 + margin), row * (1 + margin), 0);
            i++;
        }
    }
}
const loadAnalyse = () => {
    hideGroup([galleryGroup, extraGroup])
    analyseGroup.children.forEach(child => {
        analyseGroup.remove(child)
        //TODO: dispose everything from child

    })
    if (activeImage.value) {
        analyseGroup.add(activeImage.value?.clone())
        analyseGroup.children[0].visible = true
        moveCameraToPosition(activeImage.value.position)
    }


}

const lastCameraPosition = new THREE.Vector3()
const lastCameraTarget = new THREE.Vector3()
const moveCameraToPosition = (position: THREE.Vector3) => {
    lastCameraPosition.copy(camera.position)
    lastCameraTarget.copy(controls.target)
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
const loadExtra = () => {
    sceneMode.value = 'generate'
    const geometry = new THREE.BoxGeometry(1, 1, 1);
    const material = new THREE.MeshBasicMaterial({ color: 0x0000ff });
    const cube = new THREE.Mesh(geometry, material);

    extraGroup.add(cube);
    hideGroup([galleryGroup, analyseGroup])
}
const onDrag = () => {
    if (isMouseDown.value) {
        isDragging.value = true
    }
}
const mouseUp = () => {
    isMouseDown.value = false
    setTimeout(() => {
        isDragging.value = false
    }, 100);
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
            console.log(object.userData.image)
            activeImage.value = object
            sceneMode.value = 'analyse'
        }
    }
}

const addLine2 = (points: number[]) => {
    const geometry = new LineGeometry();
    geometry.setPositions(points);

    const material = new LineMaterial({
        color: 0xffffff,
        linewidth: 5, // in pixels
    });

    const line = new Line2(geometry, material);
    line.computeLineDistances();
    line.scale.set(1, 1, 1);

    return line;
};

const showHoughLines = async () => {
    console.log('show hough lines')
    console.log(activeImage.value?.userData.image.file_name)
    const image = activeImage.value?.userData.image;
    //console.log('image:', activeImage.value?.userData.image.resolution.width);    
    console.log('userdata:', activeImage.value?.userData);

    const imagePosition = activeImage.value?.position; // Get the position of the image plane
    console.log('image position:', imagePosition);

    // Get image dimensions
    //const imageWidth = image.resolution.width;
    const imageHeight = image.resolution.height;

    // fetch hough lines from json file 
    const folderName = 'deep_hough_10k_orig01_out01_jsons'
    const fileName = activeImage.value?.userData.image.file_name.split('.')[0]
    const points: number[][] = [];
    await fetch(`/${folderName}/${fileName}.json`)
        .then(response => response.json())
        .then(data => {
            console.log(data)
            data.forEach((line: any) => {
                // points.push([
                //      line.y1 / imageWidth-0.5+imagePosition.x, (imageHeight - line.x1) / imageHeight-0.5+imagePosition.y, 0, 
                //      line.y2 / imageWidth-0.5+imagePosition.x, (imageHeight -line.x2) / imageHeight-0.5+imagePosition.y, 0
                // ])
                points.push([
                    line.y1 / imageHeight + (imagePosition?.x ?? 0) - 0.5, (imageHeight - line.x1) / imageHeight - 0.5 + (imagePosition?.y ?? 0), 0,
                    line.y2 / imageHeight + (imagePosition?.x ?? 0) - 0.5, (imageHeight - line.x2) / imageHeight - 0.5 + (imagePosition?.y ?? 0), 0
                ])
            });

        });

    //debugger;
    points.forEach((line: number[]) => {
        const lineObject = addLine2(line);
        console.log('line: ', line)
        scene.add(lineObject);
    });
}

</script>
<style scoped></style>