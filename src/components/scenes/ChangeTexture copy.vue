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
        <div ref="objectContainer" class="object-container"></div>

        <!-- textures controls -->
        <div ref="textureControlsContainer" class="texture-controls">
          <section class="texture-controls__section">
            <h2 class="texture-controls__title">Título section</h2>
            <div class="texture-controls__buttons">
              <button
                class="texture-controls__button"
                @click="
                  updateSofaTexture(
                    'mesh_Cadeira000_1',
                    '/assets/change_texture/textures/964.jpg'
                  )
                "
              >
                <img
                  class="texture-controls__image"
                  src="/assets/change_texture/textures/964.jpg"
                  alt="texture"
                />
                <span class="texture-controls__label">Texture 1</span>
              </button>
              <button
                class="texture-controls__button"
                @click="
                  updateSofaTexture(
                    'mesh_Cadeira004',
                    '/assets/change_texture/textures/967.jpg'
                  )
                "
              >
                <img
                  class="texture-controls__image"
                  src="/assets/change_texture/textures/967.jpg"
                  alt="texture"
                />
                <span class="texture-controls__label">Texture 2</span>
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

const isWebGL2Available = WebGL.isWebGL2Available();
const getWebGL2ErrorMessage = WebGL.getWebGL2ErrorMessage().outerHTML;

// Canvas container
const objectContainer = ref(null);
const header = ref(null);
const textureControlsContainer = ref(null);

const loading = ref(true);

// Scene
const scene = new THREE.Scene();
scene.background = new THREE.Color(0xe3e3e3);

// Camera
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  10
);

camera.position.set(0.5, 1, 1);
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

// Objects
let object = null;
const createObject = () => {
  const loader = new GLTFLoader();
  loader.load(
    "/assets/change_texture/objects/poltrona.glb",
    // "/assets/change_texture/testes/Cadeira4K.glb",
    // "/assets/change_texture/testes/Cadeira2K.glb",
    // "/assets/change_texture/testes/Cadeira1K.glb",
    // "/assets/change_texture/testes/Cadeira512.glb",
    function (gltf) {
      object = gltf.scene;
      object.name = "loadedObject";
      console.log("object", object);
      object.position.set(0, -0.2, 0);

      // object.traverse((child) => {
      //   if (child instanceof THREE.Mesh) {
      //     child.castShadow = true;
      //     child.receiveShadow = true;
      //   }
      // });

      scene.add(object);

      // updateSofaTexture("mesh_Cadeira004");
      // x3.add(sofa, { label: "Sofa" });
    },

    // called while loading is progressing
    function (xhr) {
      // console.log((xhr.loaded / xhr.total) * 100 + "% loaded");
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

const updateSofaTexture = (childName, filePath) => {
  console.log('childName', childName);
  const textureLoader = new THREE.TextureLoader();
  const newTexture = textureLoader.load(filePath);

  object.traverse((child) => {
    console.log('child name', child);
    if (child instanceof THREE.Mesh && child.name === childName) {
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
    const windowHeight = window.innerHeight;
    const headerHeight = header.value.clientHeight;

    textureControlsContainer.value.innerHeight = windowHeight - headerHeight;

    console.log("window he ", windowHeight);
    console.log(`header he`, headerHeight);
    const width = objectContainer.value.clientWidth;
    const height = windowHeight - headerHeight;

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
  padding: 20px 15px;
  background: #2a2e34;
  
}

.header__title {
  color: #ff9900;
  margin-bottom: 10px;
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
  max-width: 800px;
  height: auto;
  background-color: lightgray;
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
}

.texture-controls__section {
  display: flex;
  flex-direction: column;
  gap: 12px;
  align-items: center;
  width: 100%;
  border-bottom: 1px solid rgb(123, 123, 123);
  padding: 40px 16px;
  height: 200vh;
}

.texture-controls__title {
  color: #2a2e34;
}

.texture-controls__buttons {
  display: flex;
  gap: 12px;
}

.texture-controls__button {
  width: 100px;
  height: 100px;
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
