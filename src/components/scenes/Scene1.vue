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

const clickInTransformController = ref(false);
const currentTransformControlesMode = ref("translate");
const hasSelectedObject = ref(false);

const loading = ref(true);


// Scene
const scene = new THREE.Scene();

// Camera
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.set(2.5, 3, 2.5);
camera.lookAt(0, 0, 0);

// Renderer
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
renderer.physicallyCorrectLights = true;

//init x3
let x3 = null;
if (!window.x3) {
  x3 = new THREEx3(
    {
      THREE,
      OrbitControls,
      camera,
      renderer,
      scene,
    },
    {
      grid: { visible: false },
      axes: { visible: false },
      stats: { visible: false },
    }
  );
  window.x3 = x3;
} else {
  x3 = window.x3;
}

// Light
let lampLight1 = null;
let lampLight2 = null;
function createLights() {
  const hemiLight = new THREE.HemisphereLight(0xffffff, 0x444444, 0.2);
  lampLight1 = new THREE.PointLight(0xfffae9, 6, 100, 2);
  lampLight2 = new THREE.PointLight(0xfffae9, 6, 100, 2);

  lampLight1.castShadow = true;
  lampLight1.position.set(0, 2.7, -1);

  lampLight1.shadow.mapSize.width = 1024;
  lampLight1.shadow.mapSize.height = 1024;
  lampLight1.shadow.bias = -0.005;

  lampLight2.castShadow = true;
  lampLight2.position.set(0, 2.7, 1);

  lampLight2.shadow.mapSize.width = 1024;
  lampLight2.shadow.mapSize.height = 1024;
  lampLight2.shadow.bias = -0.005;

  scene.add(hemiLight);
  scene.add(lampLight1);
  scene.add(lampLight2);

  // x3.add(lampLight1, {
  //   label: "Lamp Light 1",
  //   visible: false,
  //   helper: { visible: true },
  // });
}
createLights();

const guiaGeometry = new THREE.BoxGeometry();
const guiaMaterial = new THREE.MeshBasicMaterial({
  color: 0xddd000,
});

// Objects

let cabinet = null;
let cabinetGuia = null;
const createCabinet = () => {
  //start guia==============
  const guia = new THREE.Mesh(guiaGeometry, guiaMaterial);
  guia.visible = false;

  guia.position.set(-2.64, 0, 1.5);
  guia.rotation.y = THREE.MathUtils.degToRad(-90);

  cabinetGuia = guia;
  cabinetGuia.name = "cabinetGuia";
  scene.add(cabinetGuia);
  // x3.add(cabinetGuia, { label: "Cabinet Giua" });
  //end guia==============

  const loader = new GLTFLoader();
  loader.load(
    "/assets/3d_objects/modern_cabinet.glb",
    function (gltf) {
      cabinet = gltf.scene;
      cabinet.name = "cabinet";

      cabinet.traverse((child) => {
        if (child instanceof THREE.Mesh) {
          child.castShadow = true;
          child.receiveShadow = true;
        }
      });

      resizeGuia(cabinet, cabinetGuia);
      cabinet.position.copy(cabinetGuia.position);
      cabinet.rotation.copy(cabinetGuia.rotation);
      scene.add(cabinet);

      // x3.add(cabinet, { label: "Cabinet" });
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

let sofa = null;
let sofaGuia = null;
const createSofa = () => {
  //start guia==============
  const guia = new THREE.Mesh(guiaGeometry, guiaMaterial);
  guia.visible = false;

  guia.position.set(0.8, 0, -1.57);
  guia.rotation.y = THREE.MathUtils.degToRad(0);
  sofaGuia = guia;
  sofaGuia.name = "sofaGuia";
  scene.add(sofaGuia);
  // x3.add(sofaGuia, { label: "Sofa Giua" });
  //end guia==============

  const loader = new GLTFLoader();
  loader.load(
    "/assets/3d_objects/sofa-aberto2.glb",
    function (gltf) {
      sofa = gltf.scene;
      sofa.name = "sofa";

      sofa.traverse((child) => {
        if (child instanceof THREE.Mesh) {
          child.castShadow = true;
          child.receiveShadow = true;
        }
      });

      resizeGuia(sofa, sofaGuia);
      sofa.position.copy(sofaGuia.position);
      sofa.rotation.copy(sofaGuia.rotation);
      scene.add(sofa);
      // x3.add(sofa, { label: "Sofa" });
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

let tvTable = null;
let tvTableGuia = null;
const createTvTable = () => {
  //start guia==============
  const guia = new THREE.Mesh(guiaGeometry, guiaMaterial);
  guia.visible = false;

  guia.position.set(-2.69, 0, -1.33);
  guia.rotation.y = THREE.MathUtils.degToRad(90);
  guia.scale.set(1.2, 1.2, 1.2);

  tvTableGuia = guia;
  tvTableGuia.name = "tvTableGuia";
  scene.add(tvTableGuia);
  // x3.add(tvTableGuia, { label: "Tv Table Giua" });
  //end guia==============

  const loader = new GLTFLoader();
  loader.load(
    "/assets/3d_objects/tv-table3.glb",
    function (gltf) {
      tvTable = gltf.scene;
      tvTable.name = "tvTable";

      tvTable.traverse((child) => {
        if (child instanceof THREE.Mesh) {
          child.castShadow = true;
          child.receiveShadow = true;
        }
      });

      resizeGuia(tvTable, tvTableGuia);
      tvTable.position.copy(tvTableGuia.position);
      tvTable.rotation.copy(tvTableGuia.rotation);
      tvTable.scale.copy(tvTableGuia.scale);
      scene.add(tvTable);
      // x3.add(tvTable, { label: "Tv Table" });
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

//ambient
const createCeiling = () => {
  const ceilingGeometry = new THREE.PlaneGeometry(8, 8);
  const ceilingMaterial = new THREE.MeshStandardMaterial({
    color: 0xffffff,
    side: THREE.DoubleSide,
  });
  const ceiling = new THREE.Mesh(ceilingGeometry, ceilingMaterial);
  ceiling.position.y = 3;
  ceiling.rotation.x = THREE.MathUtils.degToRad(90);
  scene.add(ceiling);
};

let floor = null;
const createFloor = () => {
  const textureLoader = new THREE.TextureLoader();
  const woodTexture = new THREE.MeshStandardMaterial({
    map: textureLoader.load("/assets/textures/woodfloor/Wood_Floor_012_basecolor.jpg"),
    normalMap: textureLoader.load(
      "/assets/textures/woodfloor/Wood_Floor_012_normal.jpg"
    ),
    roughnessMap: textureLoader.load(
      "/assets/textures/woodfloor/Wood_Floor_012_roughness.jpg"
    ),
    aoMap: textureLoader.load(
      "/assets/textures/woodfloor/Wood_Floor_012_ambientOcclusion.jpg"
    ),
  });

  const floorGeometry = new THREE.PlaneGeometry(6, 6);
  const floorMaterial = woodTexture;
  floor = new THREE.Mesh(floorGeometry, floorMaterial);
  floor.rotation.x = THREE.MathUtils.degToRad(-90);
  floor.receiveShadow = true;
  scene.add(floor);

  floor.userData.ground = true;
};

let wall1 = null;
let wall2 = null;
let wall3 = null;
let wall4 = null;
const createWalls = () => {
  const textureLoader = new THREE.TextureLoader();
  const innerWallTexture = new THREE.MeshStandardMaterial({
    map: textureLoader.load(
      "/assets/textures/innerWall/Wall_Interior_001_basecolor.jpg"
    ),
    normalMap: textureLoader.load(
      "/assets/textures/innerWall/Wall_Interior_001_normal.jpg"
    ),
    roughnessMap: textureLoader.load(
      "/assets/textures/innerWall/Wall_Interior_001_roughness.jpg"
    ),
    aoMap: textureLoader.load(
      "/assets/textures/innerWall/Wall_Interior_001_ambientOcclusion.jpg"
    ),
  });

  const wall1Geometry = new THREE.BoxGeometry(6, 3, 0.1);
  const wall1Material = innerWallTexture;
  // const wall1Material = new THREE.MeshToonMaterial({
  //   color: 0x2bbac2,
  //   side: THREE.DoubleSide,
  // });
  wall1 = new THREE.Mesh(wall1Geometry, wall1Material);
  wall1.position.y = 1.5;
  wall1.position.z = -3;
  wall1.receiveShadow = true;
  scene.add(wall1);

  const wall2Geometry = new THREE.BoxGeometry(6, 3, 0.1);
  const wall2Material = innerWallTexture;
  wall2 = new THREE.Mesh(wall2Geometry, wall2Material);
  wall2.position.y = 1.5;
  wall2.position.x = -3;
  wall2.rotation.y = -Math.PI / 2;
  wall2.receiveShadow = true;
  scene.add(wall2);
  // x3.add(wall2, { label: "wall2" });

  const wall3Geometry = new THREE.BoxGeometry(6, 3, 0.1);
  const wall3Material = innerWallTexture;
  wall3 = new THREE.Mesh(wall3Geometry, wall3Material);
  wall3.position.y = 1.5;
  wall3.position.x = 3;
  wall3.rotation.y = -Math.PI / 2;
  wall3.receiveShadow = true;
  scene.add(wall3);
  // x3.add(wall3, { label: "wall3" });

  const wall4Geometry = new THREE.BoxGeometry(6, 3, 0.1);
  const wall4Material = innerWallTexture;
  wall4 = new THREE.Mesh(wall4Geometry, wall4Material);
  wall4.position.y = 1.5;
  wall4.position.z = 3;

  wall4.receiveShadow = true;
  scene.add(wall4);
  // x3.add(wall4, { label: "wall4" });
};

createCabinet();
createSofa();
createTvTable();

createFloor();
createCeiling();
createWalls();

function resizeGuia(object, guia) {
  // Calculate object dimensions
  const box = new THREE.Box3().setFromObject(object);
  const size = new THREE.Vector3();
  box.getSize(size);

  // Update guia dimensions
  if (guia) {
    const geometry = new THREE.BoxGeometry(size.x, size.y, size.z);
    geometry.translate(0, size.y / 2, 0);
    guia.geometry.dispose();
    guia.geometry = geometry;
  }
}

// Controls and helpers
// Transform controls
const transformControls = new TransformControls(camera, renderer.domElement);
transformControls.size = 0.6;

transformControls.addEventListener("dragging-changed", function (event) {
  x3.orbitController.orbit.enabled = !event.value;
});

function limitPositionOnChange(object, floorSize) {
  const box = new THREE.Box3().setFromObject(object);
  const size = new THREE.Vector3();
  box.getSize(size);

  const halfSizeX = size.x / 2;
  const halfSizeZ = size.z / 2;

  object.position.x = Math.max(
    -floorSize + halfSizeX,
    Math.min(floorSize - halfSizeX, object.position.x)
  );
  object.position.z = Math.max(
    -floorSize + halfSizeZ,
    Math.min(floorSize - halfSizeZ, object.position.z)
  );
}

function updateTransformsData(object, guia) {
  object.position.copy(guia.position);
  object.rotation.copy(guia.rotation);
  object.scale.copy(guia.scale);
}

transformControls.addEventListener("objectChange", function () {
  const floorSize = 3;

  if (transformControls.object === sofaGuia) {
    limitPositionOnChange(sofaGuia, floorSize);
    updateTransformsData(sofa, sofaGuia);
  }

  if (transformControls.object === tvTableGuia) {
    limitPositionOnChange(tvTableGuia, floorSize);
    updateTransformsData(tvTable, tvTableGuia);
  }

  if (transformControls.object === cabinetGuia) {
    limitPositionOnChange(cabinetGuia, floorSize);
    updateTransformsData(cabinet, cabinetGuia);
  }
});

transformControls.addEventListener("mouseDown", function (event) {
  clickInTransformController.value = true;
});

transformControls.showY = false;
transformControls.setMode("translate");
scene.add(transformControls);

// helpers
// x3.add(camera, { open: false });
x3.orbitController.orbit.minPolarAngle = THREE.MathUtils.degToRad(0);
x3.orbitController.orbit.maxPolarAngle = THREE.MathUtils.degToRad(85);
x3.orbitController.orbit.minDistance = 0.05;
x3.orbitController.orbit.maxDistance = 3;

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
    [cabinetGuia, sofaGuia, tvTableGuia].filter((obj) => obj !== null),
    false
  );
  if (intersects.length > 0) {
    const object = intersects[0].object;
    if (object !== selectedObject) {
      selectedObject = object;
    }
  }
}

function handleChangeControl() {
  if (currentTransformControlesMode.value === "translate") {
    transformControls.setMode("rotate");
    transformControls.showY = true;
    transformControls.showZ = false;
    transformControls.showX = false;
    currentTransformControlesMode.value = "rotate";
  } else {
    transformControls.setMode("translate");
    transformControls.showY = false;
    transformControls.showZ = true;
    transformControls.showX = true;
    currentTransformControlesMode.value = "translate";
  }
}

function onPointerDown(event) {
  pointer.x = (event.clientX / window.innerWidth) * 2 - 1;
  pointer.y = -(event.clientY / window.innerHeight) * 2 + 1;
  raycaster.setFromCamera(pointer, camera);

  const intersects = raycaster.intersectObjects(
    [cabinetGuia, sofaGuia, tvTableGuia].filter((obj) => obj !== null),
    false
  );

  function toogleTransformControls() {
    if (intersects.length > 0) {
      selectedObject = intersects[0].object;
      transformControls.attach(selectedObject);
      hasSelectedObject.value = true;
    }

    if (intersects.length === 0 && !clickInTransformController.value) {
      selectedObject = null;
      transformControls.detach();
      hasSelectedObject.value = false;
    }
  }

  toogleTransformControls();
  clickInTransformController.value = false
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
  if (cabinet && sofa && tvTable) {
    loading.value = false
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
  <div class="three-cotainer" v-if="!isWebGL2Available">
    <div v-html="getWebGL2ErrorMessage"></div>
  </div>
  <template v-else>
    <div ref="scene1Container" class="scene1-container" v-show="!loading">
      <div class="content-container">
        <div class="controls-buttons" v-if="hasSelectedObject">
          <button @pointerdown.stop="handleChangeControl">
            <template v-if="currentTransformControlesMode === 'rotate'">
              <img src="/assets/icons/move.svg" alt="Mover" title="Mover">
              <span>Mover</span>
            </template>
            <template v-else>
              <img src="/assets/icons/rotate-solid.svg" alt="Girar" title="Girar">
              <span>Girar</span>
            </template>
          </button>
        </div>
      </div>
    </div>
    <div v-show="loading" class="loading-indicator">
    </div>
  </template>
</template>

<style scoped>
.scene1-container {
  width: 100%;
  height: 100vh;
  background-color: lightgray;
}

.three-cotainer {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  width: 100%;
  position: relative;
}

.content-container {
  position: absolute;
  z-index: 99;
  bottom: 15px;
  left: 50%;
  transform: translate(-50%, -50%);
}

.controls-buttons {
  width: fit-content;
  padding: 5px 10px;
  display: flex;
  align-items: center;
  gap: 15px;
}

.controls-buttons button {
  color: white;
  background: rgba(0, 0, 0, .5);
  padding: 5px;
  min-width: 50px;
  border: none;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 3px;
  cursor: pointer;
  transition: 150ms ease-in-out ;
  opacity: .7;
}

.controls-buttons button:hover {
  opacity: 1;
}

.controls-buttons button img {
  width: 20px;
}

.loading-indicator {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  width: 100%;
  background-color: rgb(51, 51, 51);
}

.loading-indicator::before {
  content: "";
  box-sizing: border-box;
  width: 40px;
  height: 40px;
  border: 4px solid rgb(255, 164, 66);
  border-top-color: transparent;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

</style>
