<script setup>
import * as THREE from "three";
import WebGL from "three/addons/capabilities/WebGL.js";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";
import { TransformControls } from "three/addons/controls/TransformControls.js";
import { onMounted, ref } from "vue";
import THREEx3 from "three-x3";
import { EffectComposer } from "three/addons/postprocessing/EffectComposer.js";
import { RenderPass } from "three/addons/postprocessing/RenderPass.js";
import { OutlinePass } from "three/addons/postprocessing/OutlinePass.js";

import { GLTFLoader } from "three/addons/loaders/GLTFLoader.js";
import { OBJLoader } from "three/addons/loaders/OBJLoader.js";

const isWebGL2Available = WebGL.isWebGL2Available();
const getWebGL2ErrorMessage = WebGL.getWebGL2ErrorMessage().outerHTML;

THREE.Object3D.DefaultUp = new THREE.Vector3(0, 0, 1);

// Canvas container
const scene1Container = ref(null);

// Scene
const scene = new THREE.Scene();

// Camera
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.set(2, 3, 0);
camera.lookAt(0, 0, 0);

// Renderer
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
renderer.physicallyCorrectLights = true;

// Composer
const composer = new EffectComposer(renderer);
const renderPass = new RenderPass(scene, camera);
composer.addPass(renderPass);

const outlinePass = new OutlinePass(
  new THREE.Vector2(window.innerWidth, window.innerHeight),
  scene,
  camera
);
composer.addPass(outlinePass);

// Init x3
const x3 = new THREEx3({ THREE, OrbitControls, camera, renderer, scene });
x3.add(camera);

// Light
const light = new THREE.PointLight(0xffffff, 10, 100);
// const light = new THREE.DirectionalLight( 0xffffff, 2 );
light.castShadow = true;
light.position.set(0.5, 3, 0.5);
scene.add(light);
x3.add(light, { label: "Light", visible: false });

// Objects
let sofa = null;

const createSofa = () => {
  // Instantiate a loader
  const loader = new GLTFLoader();

  // sofa
  loader.load(
    "/3d_objects/sofa.glb",
    function (gltf) {
      sofa = gltf.scene
      
      scene.add(sofa);

      sofa.position.y = 1;

      sofa.castShadow = true;
      sofa.receiveShadow = true;
      console.log("gltf", gltf);

      // Define userData for the main scene object
      sofa.userData.draggable = true;
      sofa.userData.name = "sofa";

      // Define userData for all child objects
      sofa.traverse((child) => {
        if (child.isMesh || child.isGroup) {
          child.userData.draggable = true;
          child.userData.name = "sofa";
        }
      });
      x3.add(sofa, { label: "sofa" });
    },

    // called while loading is progressing
    function (xhr) {
      console.log((xhr.loaded / xhr.total) * 100 + "% loaded");
    },
    // called when loading has errors
    function (error) {
      console.log("An error happened", error);
    }
  );
};

const createCube1 = () => {
  const cube1Geometry = new THREE.BoxGeometry(1, 1, 1);
  const cube1Material = new THREE.MeshToonMaterial({ color: 0x009900 });
  const cube1 = new THREE.Mesh(cube1Geometry, cube1Material);
  cube1.position.y = 0.5;
  cube1.position.x = 1.5;
  cube1.castShadow = true;
  cube1.receiveShadow = true;
  scene.add(cube1);
  x3.add(cube1, { label: "Cube1" });

  cube1.userData.draggable = true;
  cube1.userData.name = "Cube1";
};

const createCube2 = () => {
  const cube2Geometry = new THREE.BoxGeometry(2, 2, 1);
  const cube2Material = new THREE.MeshToonMaterial({ color: 0x00ffff });
  const cube2 = new THREE.Mesh(cube2Geometry, cube2Material);
  cube2.position.y = 1;
  cube2.position.x = -2;
  cube2.castShadow = true;
  cube2.receiveShadow = true;
  scene.add(cube2);
  x3.add(cube2, { label: "Cube2" });

  cube2.userData.draggable = true;
  cube2.userData.name = "Cube2";
};

const createFloor = () => {
  const floorGeometry = new THREE.PlaneGeometry(10, 10);
  const floorMaterial = new THREE.MeshToonMaterial({
    color: 0xffffff,
    side: THREE.DoubleSide,
  });
  const floor = new THREE.Mesh(floorGeometry, floorMaterial);
  floor.rotation.x = -Math.PI / 2;
  floor.receiveShadow = true;
  scene.add(floor);
  x3.add(floor, { label: "Floor" });

  floor.userData.ground = true;
};

const createWalls = () => {
  const wall1Geometry = new THREE.PlaneGeometry(10, 3);
  const wall1Material = new THREE.MeshToonMaterial({
    color: 0x2bbac2,
    side: THREE.DoubleSide,
  });
  const wall1 = new THREE.Mesh(wall1Geometry, wall1Material);
  wall1.position.y = 1.5;
  wall1.position.z = -5;
  wall1.receiveShadow = true;
  scene.add(wall1);
  x3.add(wall1, { label: "Wall1" });

  const wall2Geometry = new THREE.PlaneGeometry(10, 3);
  const wall2Material = new THREE.MeshToonMaterial({
    color: 0x2bbac2,
    side: THREE.DoubleSide,
  });
  const wall2 = new THREE.Mesh(wall2Geometry, wall2Material);
  wall2.position.y = 1.5;
  wall2.position.x = -3;
  wall2.rotation.y = -Math.PI / 2;
  wall2.receiveShadow = true;
  scene.add(wall2);
  x3.add(wall2, { label: "Wall2" });
};

createSofa();
createCube1();
createCube2();
createFloor();
createWalls();

// Helpers
const axesHelper = new THREE.AxesHelper(3);
scene.add(axesHelper);

// Raycaster
const raycaster = new THREE.Raycaster();
const clickMouse = new THREE.Vector2();
const moveMouse = new THREE.Vector2();
let draggable = null;
let hoverObject = null;

// Event listeners
window.addEventListener("click", (event) => {
  if (draggable) {
    console.log("draggable", draggable.userData.name);

    draggable = null;
    outlinePass.selectedObjects = [];
    return;
  }

  console.log("event", event);

  clickMouse.x = (event.clientX / window.innerWidth) * 2 - 1;
  clickMouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

  raycaster.setFromCamera(clickMouse, camera);
  const found = raycaster.intersectObjects(scene.children);

  console.log("found", found);

  if (found.length > 0 && found[0].object.userData.draggable) {
    draggable = found[0].object;
    console.log("click", draggable.userData.name);
    outlinePass.selectedObjects = [draggable];
  } else {
  }
});

window.addEventListener("mousemove", (event) => {
  moveMouse.x = (event.clientX / window.innerWidth) * 2 - 1;
  moveMouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

  raycaster.setFromCamera(moveMouse, camera);

  const found = raycaster.intersectObjects(scene.children);

  if (found.length > 0 && found[0].object.userData.draggable) {
    hoverObject = found[0].object;
    outlinePass.selectedObjects = [hoverObject];
  } else {
    hoverObject = null;
    outlinePass.selectedObjects = [];
  }

  if (draggable) {
    const found = raycaster.intersectObjects([
      scene.children.filter((obj) => obj.userData.ground)[0],
    ]);
    draggable.position.x = found[0].point.x;
    draggable.position.z = found[0].point.z;
  }
});

function animate() {
  x3.tick();
  x3.fps(() => {
    composer.render();
  });

  composer.render();
}

onMounted(() => {
  if (scene1Container.value) {
    scene1Container.value.appendChild(renderer.domElement);
    renderer.setAnimationLoop(animate);
  }
});
</script>

<template>
  <div v-if="!isWebGL2Available">
    <div v-html="getWebGL2ErrorMessage"></div>
  </div>
  <div v-else ref="scene1Container" class="scene1-container"></div>
</template>

<style scoped>
.scene1-container {
  width: 100%;
  height: 100vh;
  background-color: lightgray;
}
</style>
