<script setup lang="ts">
import {
  AxesHelper,
  DirectionalLight,
  Fog,
  HemisphereLight,
  Mesh,
  PerspectiveCamera,
  Scene,
  Vector3,
  WebGLRenderer,
} from "three";
import {
  OrbitControls,
  ParametricGeometry,
} from "three/examples/jsm/Addons.js";

let camera: PerspectiveCamera, scene: Scene, renderer: WebGLRenderer;
let controls: OrbitControls;

var text = "Happy Birthday! Dear Franciscah! --- ^o^ --- ";
var textColors = [0x00bfff, 0xff00ff, 0x7cfc00, 0xdc143c, 0x7fff00, 0x00bfff, 0x00ffff,0xff0000];
var textScaleSize = 0.3; //delta scale for letters
var textAnimeColors = []; // Index of current color for each letter
var textRotSpeed = 0.2;  // Text world rotation speed. in degree
var textInitialSize = 120; // Initial font size for text
var textYScope = 5  // Text Sin wave height.
var textYSpeed = 10  // Text Sin wave speed
var textBevelThickness = 3; // Text bevel thickness
var textBevelSize = 3;   // Text bevel size
var textFont = "fonts/gentilis_bold.typeface.json";  // textFont  ::Only ThreeJS fonts are available
var distanceToText = 1000; // Distance from cakemodel to text
var textSizeInAngle = 8; // Angle width for each letter.
var textRot = 0;  // current text rotation.
var textColorTransformSpeed = 50; // text color transform speed.  !CAUTION: The smaller the faster
var textObjects = [];

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

    camera = new PerspectiveCamera(30, offsetWidth / offsetHeight, 0.1, 1000);
    camera.position.set(
      Math.cos(Math.PI / 4) * 3,
      0,
      Math.sin(Math.PI / 4) * 3
    );

    controls = new OrbitControls(camera, container);
    // controls.maxDistance = 9;
    // controls.maxPolarAngle = MathUtils.degToRad(90);
    controls.target.set(0, 0, 0);
    controls.update();

    scene = new Scene();
    // scene.background = new Color(0xf9fafb);
    scene.fog = new Fog(0x00ff00, 500, 1000);

    const directionalLight = new DirectionalLight(0xffffff, 1.75);
    const d = 5;
    directionalLight.position.set(d, d, d);
    directionalLight.castShadow = true;
    // directionalLight.shadowMapWidth = 1024 * 2;
    // directionalLight.shadowMapHeight = 1024 * 2;

    // directionalLight.shadowCameraLeft = -d;
    // directionalLight.shadowCameraRight = d;
    // directionalLight.shadowCameraTop = d;
    // directionalLight.shadowCameraBottom = -d;

    // directionalLight.shadowCameraFar = 3 * d;
    // directionalLight.shadowCameraNear = d;
    // directionalLight.shadowDarkness = 0.5;
    scene.add(directionalLight);

    const hemishpericalLight = new HemisphereLight(0x666666, 0x080820, 1);
    scene.add(hemishpericalLight);

    const axesHelper = new AxesHelper(5);
    scene.add(axesHelper);
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
  <div class="relative h-full">
    <div class="absolute inset-0" ref="canvasContainer"></div>
  </div>
</template>
