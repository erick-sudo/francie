<script setup lang="ts">
import {
  Color,
  GridHelper,
  MathUtils,
  PerspectiveCamera,
  Scene,
  WebGLRenderer,
} from "three";
import { OrbitControls, RGBELoader } from "three/examples/jsm/Addons.js";

let camera: PerspectiveCamera, scene: Scene, renderer: WebGLRenderer;
let gridHelper: GridHelper;
let controls: OrbitControls;

const canvasContainer = useTemplateRef("canvasContainer");

function init() {
  if (canvasContainer.value) {
    const container = canvasContainer.value;
    const { offsetWidth, offsetHeight } = container;

    renderer = new WebGLRenderer({ antialias: true });
    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.setSize(offsetWidth, offsetHeight);
    renderer.setAnimationLoop(animate);

    Array.from(container.children).forEach((child) => child.remove());

    container.appendChild(renderer.domElement);

    window.addEventListener("resize", onWindowResize);
    //

    camera = new PerspectiveCamera(40, offsetWidth / offsetHeight, 0.1, 100);
    camera.position.set(4.25, 1.4, -4.5);

    controls = new OrbitControls(camera, container);
    controls.maxDistance = 9;
    controls.maxPolarAngle = MathUtils.degToRad(90);
    controls.target.set(0, 0.5, 0);
    controls.update();

    scene = new Scene();
    scene.background = new Color(0xf9fafb);

    gridHelper = new GridHelper(20, 40, 0x333333, 0x333333);
    // gridHelper.material.opacity = 0.2;
    // gridHelper.material.depthWrite = false;
    // gridHelper.material.transparent = true;
    scene.add(gridHelper);
    
  }
}

function onWindowResize() {
  if (canvasContainer.value && canvasContainer.value.parentElement) {
    const { offsetWidth, offsetHeight } = canvasContainer.value.parentElement;

    camera.aspect = offsetWidth / offsetHeight;
    camera.updateProjectionMatrix();

    renderer.setSize(offsetWidth, offsetHeight);
  }
}

function animate() {
  controls.update();
  const time = -performance.now() / 1000;

  renderer.render(scene, camera);
}

onMounted(() => {
  init();
});
</script>

<template>
  <div ref="canvasContainer"></div>
</template>
