<template>
  <div class="container">
    <header class="header" ref="header">
      <h1 class="header__title">Objeto 3D - Texturas</h1>
      <p class="header__description">
        Selecione a textura para atualizar o objeto 3d.
      </p>
    </header>

    <!-- loading -->
    <div v-show="loading" class="loading-indicator"></div>

    <!-- webgl2 not available -->
    <div class="not-available-message" v-if="!isWebGL2Available">
      <div v-html="getWebGL2ErrorMessage"></div>
    </div>

    <template v-else>
      <div class="content-container">
        <!-- object area -->

        <div class="bg-object-container"></div>
        <div ref="objectContainer" class="object-container"></div>

        <!-- textures controls -->
        <div ref="textureControlsContainer" class="texture-controls">
          <section
            class="texture-controls__section"
            v-for="(section, index) in sections"
            :key="index"
          >
            <h2 class="texture-controls__title">{{ section.title }}</h2>
            <div class="texture-controls__buttons">
              <button
                v-for="(texture, index) in section.textures"
                :key="index"
                class="texture-controls__button"
                @click="updateSofaTexture(section.objectChildName, texture)"
              >
                <img
                  class="texture-controls__image"
                  :src="texture"
                  alt="texture"
                />
                <span class="texture-controls__label">Texture 1</span>
              </button>
            </div>
          </section>
        </div>
      </div>
    </template>
  </div>
</template>

<script setup>
import * as THREE from "three";
import WebGL from "three/addons/capabilities/WebGL.js";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";
import { TransformControls } from "three/addons/controls/TransformControls.js";
import { onMounted, ref } from "vue";
import THREEx3 from "three-x3";
import { GLTFLoader } from "three/addons/loaders/GLTFLoader.js";
import { RGBELoader } from "three/addons/loaders/RGBELoader.js";

const isWebGL2Available = WebGL.isWebGL2Available();
const getWebGL2ErrorMessage = WebGL.getWebGL2ErrorMessage().outerHTML;

const sections = [
  {
    title: "Encosto",
    objectChildName: "mesh_cadeira_almofada",
    textures: [
      // "/change_texture/textures/964.jpg",
      // "/change_texture/textures/967.jpg",
      "https://planmkt.com.br/clientes/plan/textura3d/change_texture/textures/964.jpg",
      "https://planmkt.com.br/clientes/plan/textura3d/change_texture/textures/967.jpg",
    ],
  },
  {
    title: "Assento",
    objectChildName: "mesh_Cadeira004",
    textures: [
      // "/change_texture/textures/964.jpg",
      // "/change_texture/textures/967.jpg",
      "https://planmkt.com.br/clientes/plan/textura3d/change_texture/textures/964.jpg",
      "https://planmkt.com.br/clientes/plan/textura3d/change_texture/textures/967.jpg",
    ],
  },
  {
    title: "Pés",
    objectChildName: "mesh_Cadeira002",
    textures: [
      // "/change_texture/textures/964.jpg",
      // "/change_texture/textures/967.jpg",
      "https://planmkt.com.br/clientes/plan/textura3d/change_texture/textures/964.jpg",
      "https://planmkt.com.br/clientes/plan/textura3d/change_texture/textures/967.jpg",
    ],
  },
  {
    title: "Braços",
    objectChildName: "mesh_Cadeira000",
    textures: [
      // "/change_texture/textures/964.jpg",
      // "/change_texture/textures/967.jpg",
      "https://planmkt.com.br/clientes/plan/textura3d/change_texture/textures/964.jpg",
      "https://planmkt.com.br/clientes/plan/textura3d/change_texture/textures/967.jpg",
    ],
  },
  {
    title: "Lateral",
    objectChildName: "mesh_Cadeira003",
    textures: [
      // "/change_texture/textures/964.jpg",
      // "/change_texture/textures/967.jpg",
      "https://planmkt.com.br/clientes/plan/textura3d/change_texture/textures/964.jpg",
      "https://planmkt.com.br/clientes/plan/textura3d/change_texture/textures/967.jpg",
    ],
  },
];

// Canvas container
const objectContainer = ref(null);
const header = ref(null);

const loading = ref(true);

// Scene
const scene = new THREE.Scene();
scene.background = new THREE.Color(0xffffff);

// Camera
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  10
);

camera.position.set(-0.72, 0.87, 1);
camera.lookAt(0, 0, 0);

// Renderer
const renderer = new THREE.WebGLRenderer({ antialias: true });
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
  console.log("x3", x3);
} else {
  x3 = window.x3;
  console.log("x3", x3);
}

// x3.add(camera, {
//   label: "camera",
//   visible: false,
//   helper: { visible: true },
// });

// Light
let lampLight1 = null;
let lampLight2 = null;
function createLights() {
  //Create a DirectionalLight and turn on shadows for the light
  // const directionlLight1 = new THREE.DirectionalLight(0xffffff, 5);
  // directionlLight1.position.set(3.22, 1.195, 1.143);
  // directionlLight1.rotation.set(16.83, 250.364, -1.527);
  // directionlLight1.castShadow = true;
  // scene.add(directionlLight1);

  // const directionlLight2 = new THREE.DirectionalLight(0xffffff, 5);
  // directionlLight2.position.set(-6.538, 3.012, -5.833);
  // directionlLight2.rotation.set(16.717, 47.85, 39.388);
  // directionlLight2.castShadow = true;
  // scene.add(directionlLight2);

  // const directionlLight3 = new THREE.DirectionalLight(0xffffff, 5);
  // directionlLight3.position.set(-4.362, 3.576, 2.268);
  // directionlLight3.rotation.set(30.702, 118.38, -2.062);
  // directionlLight3.castShadow = true;
  // directionlLight3.shadow.radius = 4;

  // scene.add(directionlLight3);

  const hemiLight = new THREE.HemisphereLight(0xffffff, 0xffffff, 3);
  hemiLight.position.set(0, 20, 0);
  scene.add(hemiLight);
  console.log("hemiLight", hemiLight);

  const dirLight = new THREE.DirectionalLight(0xffffff, 3);
  dirLight.position.set(3, 10, 10);
  dirLight.castShadow = true;
  dirLight.shadow.camera.top = 2;
  dirLight.shadow.camera.bottom = -2;
  dirLight.shadow.camera.left = -2;
  dirLight.shadow.camera.right = 2;
  dirLight.shadow.camera.near = 0.1;
  dirLight.shadow.camera.far = 40;
  scene.add(dirLight);
  console.log("dirLight", dirLight);

  const dirLight2 = new THREE.DirectionalLight(0xffffff, 3);
  dirLight2.position.set(-12, 10, 10);
  dirLight2.castShadow = true;
  dirLight2.shadow.camera.top = 2;
  dirLight2.shadow.camera.bottom = -2;
  dirLight2.shadow.camera.left = -2;
  dirLight2.shadow.camera.right = 2;
  dirLight2.shadow.camera.near = 0.1;
  dirLight2.shadow.camera.far = 40;
  scene.add(dirLight2);
  console.log("dirLight2", dirLight2);

  const dirLight3 = new THREE.DirectionalLight(0xffffff, 3);
  dirLight3.position.set(-6.51, 10, -5.21);
  dirLight3.castShadow = true;
  dirLight3.shadow.camera.top = 2;
  dirLight3.shadow.camera.bottom = -2;
  dirLight3.shadow.camera.left = -2;
  dirLight3.shadow.camera.right = 2;
  dirLight3.shadow.camera.near = 0.1;
  dirLight3.shadow.camera.far = 40;
  scene.add(dirLight3);
  console.log("dirLight3", dirLight3);

  x3.add(dirLight, {
    label: "dirLight",
    visible: true,
    helper: { visible: false, color: 0xffff00 },
  });
  x3.add(dirLight2, {
    label: "dirLight2",
    visible: true,
    helper: { visible: false, color: 0xffff00 },
  });
  x3.add(dirLight3, {
    label: "dirLight3",
    visible: true,
    helper: { visible: false, color: 0xffff00 },
  });

  // console.log("directionlLight1", directionlLight1);
  // console.log("directionlLight2", directionlLight2);
  // console.log("directionlLight3", directionlLight3);

  // const hemiLight = new THREE.HemisphereLight(0xffffff, 0x444444, 0.2);
  lampLight1 = new THREE.PointLight(0xfffae9, 2, 0, 2);
  lampLight2 = new THREE.PointLight(0xfffae9, 6, 100, 2);

  lampLight1.castShadow = true;
  lampLight1.position.set(0.7, 1.45, 1.52);

  lampLight1.shadow.mapSize.width = 1024;
  lampLight1.shadow.mapSize.height = 1024;
  lampLight1.shadow.bias = -0.005;
  lampLight1.radius = 10;

  lampLight2.castShadow = true;
  lampLight2.position.set(-0.7, 1.45, 1.52);

  lampLight2.shadow.mapSize.width = 1024;
  lampLight2.shadow.mapSize.height = 1024;
  lampLight2.shadow.bias = -0.005;
  lampLight2.radius = 10;
  // console.log("lampLight1", lampLight1);

  // scene.add(hemiLight);
  // scene.add(lampLight1);
  // scene.add(lampLight2);

  // x3.add(directionlLight1, {
  //   label: "directionlLight",
  //   visible: true,
  //   helper: { visible: true, color: 0xffff00 },
  // });
  // x3.add(directionlLight2, {
  //   label: "directionlLight",
  //   visible: true,
  //   helper: { visible: true, color: 0xffff00 },
  // });
  // x3.add(directionlLight3, {
  //   label: "directionlLight",
  //   visible: true,
  //   helper: { visible: true, color: 0xffff00 },
  // });
  // x3.add(lampLight1, {
  //   label: "Lamp Light 1",
  //   visible: true,
  //   helper: { visible: true, radius: 0.1 },
  // });
  // x3.add(lampLight2, {
  //   label: "Lamp Light 2",
  //   visible: false,
  //   helper: { visible: true },
  // });
}
createLights();

// Load HDR environment map
// new RGBELoader()
//   .load('/assets/change_texture/testes/SGS_AmbientMap_Automotive_1.hdr', (texture) => {
//     texture.mapping = THREE.EquirectangularReflectionMapping;
//     scene.environment = texture;
//   });

// Objects
let object = null;
const createObject = () => {
  const loader = new GLTFLoader();
  loader.load(
    // "/change_texture/objects/poltrona.glb",
    // "/change_texture/testes/Cadeira4K.glb",
    // "/change_texture/testes/Cadeira2K.glb",
    // "/change_texture/testes/v1_Cadeira1K.glb",
    // "/change_texture/testes/v1_Cadeira2K.glb",
    // "/change_texture/testes/v2_Cadeira1K.glb",
    "https://planmkt.com.br/clientes/plan/textura3d/change_texture/testes/v2_Cadeira1K.glb",
    // "/change_texture/testes/Cadeira512.glb",
    function (gltf) {
      object = gltf.scene;
      object.name = "loadedObject";
      object.position.set(0, -0.2, 0);
      object.castShadow = true;
      object.receiveShadow = true;
      object.traverse((child) => {
        if (child instanceof THREE.Mesh) {
          child.castShadow = true;
          child.receiveShadow = true;
        }
      });

      scene.add(object);
    },

    // called while loading is progressing
    function (xhr) {
      console.log((xhr.loaded / xhr.total) * 100 + "% loaded");
      if (xhr.loaded === xhr.total) {
        loading.value = false;
      }
    },
    // called when loading has errors
    function (error) {
      console.log("An error happened in sofa loader", error);
    }
  );
};

// Objects
const updateSofaTexture = (childName, filePath) => {
  // console.log('childName====>', childName);

  const textureLoader = new THREE.TextureLoader();
  const newTexture = textureLoader.load(filePath);

  object.traverse((child) => {
    console.log("child.name", child.name);
    if (child instanceof THREE.Mesh && child.userData.name === childName) {
      console.log("child name", child);
      const newMaterial = child.material.clone();
      newMaterial.map = newTexture;
      newMaterial.needsUpdate = true;
      child.material = newMaterial;
      console.log(`Textura atualizada para ${childName}`);
      console.log("child", child);
    }
  });
};

createObject();

let floor = null;
const createFloor = () => {
  const floorGeometry = new THREE.PlaneGeometry(100, 100);

  const floorMaterial = new THREE.MeshPhongMaterial({
    color: 0xcbcbcb,
    depthWrite: false,
  });
  // const floorMaterial = new THREE.MeshStandardMaterial({ color: 0xffffff });
  floor = new THREE.Mesh(floorGeometry, floorMaterial);
  floor.position.set(0, -0.2, 0);
  floor.rotation.x = THREE.MathUtils.degToRad(-90);
  floor.receiveShadow = true;
  scene.add(floor);

  floor.userData.ground = true;
};

createFloor();

x3.orbitController.orbit.minDistance = 1;
x3.orbitController.orbit.maxDistance = 4;

// Controls and helpers
// Transform controls

// helpers
// x3.add(camera, { open: false });
// x3.orbitController.orbit.minPolarAngle = THREE.MathUtils.degToRad(0);
// x3.orbitController.orbit.maxPolarAngle = THREE.MathUtils.degToRad(85);
// x3.orbitController.orbit.minDistance = 0.05;
// x3.orbitController.orbit.maxDistance = 3;

// function onWindowResize() {
//   camera.aspect = window.innerWidth / window.innerHeight;
//   camera.updateProjectionMatrix();

//   renderer.setSize(window.innerWidth, window.innerHeight);

//   render();
// }

function onWindowResize() {
  if (objectContainer.value) {
    const width = objectContainer.value.clientWidth;
    const height = objectContainer.value.clientHeight;

    camera.aspect = width / height;
    camera.updateProjectionMatrix();

    renderer.setSize(width, height);
  }

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

// function addEventListeners() {
//   if (cabinet && sofa && tvTable) {
//     loading.value = false
//     window.addEventListener("pointermove", onPointerMove);
//     window.addEventListener("pointerdown", onPointerDown);
//
//   }
// }

onMounted(() => {
  if (objectContainer.value) {
    const width = objectContainer.value.clientWidth;
    const height = objectContainer.value.clientHeight;

    console.log("width", width, "height", height);
    renderer.setSize(width, height);
    objectContainer.value.appendChild(renderer.domElement);
    renderer.setAnimationLoop(animate);
    window.addEventListener("resize", onWindowResize);

    onWindowResize();
  }
});
</script>

<style scoped>
.container {
  font-family: sans-serif;
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 100vh;
  width: 100%;
}

.header {
  width: 100%;
  text-align: center;
  background: #2a2e34;
  height: 90px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 0 12px;
}

.header__title {
  color: #ff9900;
  margin-bottom: 10px;
  font-size: 1.5rem;
}

.header__description {
  color: white;
}

.content-container {
  flex: 1;
  display: flex;
  width: 100%;
  max-width: 1920px;
  height: auto;
}

.object-container {
  width: 100%;
  max-width: 900px;
  height: calc(100vh - 90px);
  background-color: lightgray;
  display: flex;
  align-items: center;
  box-shadow: 5px 0 5px rgba(0, 0, 0, 0.02);
}

@media screen and (max-width: 1024px) {
  .content-container {
    flex-direction: column;
  }

  .object-container {
    max-width: 100%;
    height: 50vh;
    box-shadow: 0px 5 5px rgba(0, 0, 0, 0.02);
  }
}
.not-available-message {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  width: 100%;
  position: relative;
}

.texture-controls {
  display: flex;
  flex-direction: column;
  align-items: center;
  flex: 1;
  overflow-y: auto;
  height: calc(100vh - 90px);
}

.texture-controls__section {
  display: flex;
  flex-direction: column;
  gap: 12px;
  align-items: center;
  width: 100%;
  border-bottom: 1px solid rgb(123, 123, 123);
  padding: 40px 16px;
}

.texture-controls__title {
  color: #2a2e34;
  font-weight: 500;
}

.texture-controls__buttons {
  display: flex;
  gap: 12px;
}

.texture-controls__button {
  width: 80px;
  height: 80px;
  overflow: hidden;
  cursor: pointer;
  border: none;
  border-radius: 100%;
}

.texture-controls__image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: 200ms ease-in-out;
  scale: 2;
}

.loading-indicator {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  width: 100%;
  background-color: rgb(51, 51, 51);
  position: fixed;
  top: 0;
  left: 0;
  z-index: 999;
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
