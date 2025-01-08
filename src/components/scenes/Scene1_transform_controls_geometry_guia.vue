<script setup>
import * as THREE from "three";
import WebGL from "three/addons/capabilities/WebGL.js";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";
import { TransformControls } from "three/addons/controls/TransformControls.js";
import { onMounted, ref } from "vue";
import THREEx3 from "three-x3";
import { GLTFLoader } from "three/addons/loaders/GLTFLoader.js";

const isWebGL2Available = WebGL.isWebGL2Available();
const getWebGL2ErrorMessage = WebGL.getWebGL2ErrorMessage().outerHTML;

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

// Light
// const light = new THREE.PointLight(0xffffff, 10, 100);
const light = new THREE.DirectionalLight(0xffffff, 2);
// const light = new THREE.AmbientLight( 0x404040 );
light.castShadow = true;
light.position.y = 3;
light.position.x = 0.5;
light.position.z = 0.5;
scene.add(light);

//init x3
const x3 = new THREEx3({ THREE, OrbitControls, camera, renderer, scene });

// Objects
let sofa = null;
const createSofa = () => {
  // Instantiate a loader
  const loader = new GLTFLoader();

  // sofa
  loader.load(
    "/3d_objects/sofa.glb",
    function (gltf) {
      sofa = gltf.scene;

      scene.add(sofa);

      sofa.position.y = 0.17;
      sofa.position.x = 2;

      sofa.castShadow = true;
      sofa.receiveShadow = true;

      transformControls.attach(sofa);
      x3.add(sofa, { label: "sofa" });
      console.log("sofa", sofa);

      addEventListeners();
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

let roundTable = null;
const createRoundTable = () => {
  // Instantiate a loader
  const loader = new GLTFLoader();

  // table
  loader.load(
    "/3d_objects/classic_round_side_table.glb",
    function (gltf) {
      roundTable = gltf.scene;
      // scene.add(roundTable);

      roundTable.position.y = 0;
      roundTable.position.z = 0;

      roundTable.castShadow = true;
      roundTable.receiveShadow = true;

      transformControls.attach(roundTable);
      x3.add(roundTable, { label: "roundTable" });
      console.log("roundTable", roundTable);

      // Calculate dimensions
      const box = new THREE.Box3().setFromObject(roundTable);
      const size = new THREE.Vector3();
      box.getSize(size);
      console.log("Dimensions of roundTable:", size);

      const boxHelper = new THREE.BoxHelper(roundTable, 0xff0000);
      scene.add(boxHelper);

      if (cube1) {
        const geometry = new THREE.BoxGeometry(size.x, size.y, size.z);
        geometry.translate(0, size.y / 2, 0);
        cube1.geometry.dispose();
        cube1.geometry = geometry;
        cube1.position.copy(roundTable.position);
      }

      addEventListeners();
      createGroup();
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

let cube1 = null;
const createCube1 = () => {
  const cube1Geometry = new THREE.BoxGeometry(1, 1, 1);
  const cube1Material = new THREE.MeshToonMaterial({
    color: 0x009900,
    transparent: true,
    opacity: 0.5,
  });
  cube1 = new THREE.Mesh(cube1Geometry, cube1Material);
  cube1.position.y = 0.5;
  cube1.position.x = 0;
  cube1.castShadow = true;
  cube1.receiveShadow = true;
  // scene.add(cube1);
  x3.add(cube1, { label: "Cube1" });

  cube1.userData.draggable = true;
  cube1.userData.name = "Cube1";
};

let cube2 = null;
const createCube2 = () => {
  const cube2Geometry = new THREE.BoxGeometry(2, 2, 1);
  const cube2Material = new THREE.MeshToonMaterial({ color: 0x00ffff });
  cube2 = new THREE.Mesh(cube2Geometry, cube2Material);
  cube2.position.y = 1;
  cube2.position.x = -2;
  cube2.castShadow = true;
  cube2.receiveShadow = true;
  scene.add(cube2);
  x3.add(cube2, { label: "Cube2" });

  cube2.userData.draggable = true;
  cube2.userData.name = "Cube2";
};

let group = null;
function createGroup() {
  group = new THREE.Group();
  group.add(cube1);
  group.add(roundTable);
  scene.add(group);

  x3.add(group, { label: "Group" });
  console.log("group", group);
}

let floor = null;
const createFloor = () => {
  const floorGeometry = new THREE.PlaneGeometry(10, 10);
  const floorMaterial = new THREE.MeshToonMaterial({
    color: 0xffffff,
    side: THREE.DoubleSide,
  });
  floor = new THREE.Mesh(floorGeometry, floorMaterial);
  floor.rotation.x = -Math.PI / 2;
  floor.receiveShadow = true;
  scene.add(floor);
  x3.add(floor, { label: "Floor" });

  floor.userData.ground = true;
};

let wall1 = null;
let wall2 = null;
const createWalls = () => {
  const wall1Geometry = new THREE.PlaneGeometry(10, 3);
  const wall1Material = new THREE.MeshToonMaterial({
    color: 0x2bbac2,
    side: THREE.DoubleSide,
  });
  wall1 = new THREE.Mesh(wall1Geometry, wall1Material);
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
  wall2 = new THREE.Mesh(wall2Geometry, wall2Material);
  wall2.position.y = 1.5;
  wall2.position.x = -3;
  wall2.rotation.y = -Math.PI / 2;
  wall2.receiveShadow = true;
  scene.add(wall2);
  x3.add(wall2, { label: "Wall2" });
};

createSofa();
createRoundTable();
createCube1();
createCube2();
createFloor();
createWalls();

// Controls and helpers
// Transform controls
const transformControls = new TransformControls(camera, renderer.domElement);
transformControls.addEventListener("dragging-changed", function (event) {
  x3.orbitController.orbit.enabled = !event.value;
});

transformControls.addEventListener("objectChange", function () {
  if (transformControls.object === cube1) {
    roundTable.position.copy(cube1.position);
    roundTable.rotation.copy(cube1.rotation);
    roundTable.scale.copy(cube1.scale);
  }
});

transformControls.showY = false;
transformControls.setMode("translate");
scene.add(transformControls);

// helpers
const axesHelper = new THREE.AxesHelper(3);
scene.add(axesHelper);

x3.add(light, { label: "Light", visible: false });
x3.add(camera);
x3.add(cube1, { label: "Cube1" });
x3.add(cube2, { label: "Cube2" });
x3.orbitController.orbit.minPolarAngle = 0;
x3.orbitController.orbit.maxPolarAngle = Math.PI / 2;

// Raycaster
const raycaster = new THREE.Raycaster();
const pointer = new THREE.Vector2();
let selectedObject = null;

// Event listeners
function onPointerMove(event) {
  pointer.x = (event.clientX / window.innerWidth) * 2 - 1;
  pointer.y = -(event.clientY / window.innerHeight) * 2 + 1;

  raycaster.setFromCamera(pointer, camera);
  const intersects = raycaster.intersectObjects(
    [cube2, sofa, cube1].filter((obj) => obj !== null),
    false
  );
  // console.log("cube1", cube1);
  // console.log("cube2", cube2);
  // console.log("sofa", sofa.children[0]);
  // console.log("roundTable", roundTable.children[0]);
  // transformControls.attach(sofa);
  if (intersects.length > 0) {
    const object = intersects[0].object;
    if (object !== selectedObject) {
      console.log("object", object);
      selectedObject = object;
      transformControls.attach(selectedObject);
    }
  } else {
    selectedObject = null;
    transformControls.detach();
  }
}

function onPointerDown(event) {
  pointer.x = (event.clientX / window.innerWidth) * 2 - 1;
  pointer.y = -(event.clientY / window.innerHeight) * 2 + 1;

  raycaster.setFromCamera(pointer, camera);
  const intersects = raycaster.intersectObjects(
    [cube2, sofa, cube1].filter((obj) => obj !== null),
    false
  );

  if (intersects.length > 0) {
    selectedObject = intersects[0].object;
    console.log("selectedObject", selectedObject);
    transformControls.attach(selectedObject);
  } else {
    selectedObject = null;
    transformControls.detach();
  }
}

function onWindowResize() {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();

  renderer.setSize(window.innerWidth, window.innerHeight);

  render();
}

function render() {
  renderer.render(scene, camera);
}

function animate() {
  x3.tick();
  x3.fps(() => {
    renderer.render(scene, camera);
  });

  renderer.render(scene, camera);
}

function addEventListeners() {
  if (sofa && roundTable) {
    window.addEventListener("pointermove", onPointerMove);
    window.addEventListener("pointerdown", onPointerDown);
    window.addEventListener("resize", onWindowResize);
  }
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
