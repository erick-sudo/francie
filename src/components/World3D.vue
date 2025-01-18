<script setup lang="ts">
import {
  AxesHelper,
  Color,
  DirectionalLight,
  GridHelper,
  HemisphereLight,
  Mesh,
  MeshStandardMaterial,
  PerspectiveCamera,
  Scene,
  WebGLRenderer,
} from "three";
import {
  FontLoader,
  OrbitControls,
  TextGeometry,
} from "three/examples/jsm/Addons.js";

let camera: PerspectiveCamera, scene: Scene, renderer: WebGLRenderer;
let gridHelper: GridHelper;
let controls: OrbitControls;
let letters: Mesh[] = [];

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

    camera = new PerspectiveCamera(45, offsetWidth / offsetHeight, 0.1, 1000);
    camera.position.set(4.5, 10, 20);

    controls = new OrbitControls(camera, container);
    // controls.maxDistance = 9;
    // controls.maxPolarAngle = MathUtils.degToRad(90);
    controls.target.set(0, 0, 0);
    controls.update();

    scene = new Scene();
    scene.background = new Color(0xf9fafb);

    const directionalLight = new DirectionalLight("white", 1);
    directionalLight.position.set(0, 1000, 0);
    scene.add(directionalLight);

    const hemishpericalLight = new HemisphereLight(0xffffbb, 0x080820, 1);
    scene.add(hemishpericalLight);

    gridHelper = new GridHelper(100, 50, 0x333333, 0x333333);
    // gridHelper.material.opacity = 0.2;
    // gridHelper.material.depthWrite = false;
    // gridHelper.material.transparent = true;
    scene.add(gridHelper);

    const axesHelper = new AxesHelper(5);
    scene.add(axesHelper);

    const loader = new FontLoader();

    loader.load(
      "/fonts/Playwrite VN Thin_Regular.json",
      //   "/fonts/Cedarville Cursive_Regular.json",
      //   "/fonts/gentilis_regular.typeface.json",
      function (font) {
        const letterMeshes = "Happy Birthday"
          .toLowerCase()
          //   .toUpperCase()
          .split("")
          .map((letter, index) => {
            const textGeometry = new TextGeometry(letter, {
              font: font,
              size: 6,
              depth: 0.5,
              curveSegments: 32,
              bevelEnabled: true,
              bevelThickness: 0.8,
              bevelSize: 0.5,
              bevelOffset: 0,
              bevelSegments: 12,
            });
            const textMaterial = new MeshStandardMaterial({
              color: "orange",
              roughness: 0.5,
              // wireframe: true,
            });
            const textMesh = new Mesh(textGeometry, textMaterial);
            textMesh.position.set(0, 0, index * 4);
            letters.push(textMesh);
            return textMesh;
          });

        scene.add(...letterMeshes);
      }
    );
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

  letters.forEach((letter) => {
    letter.rotateY(0.01 * time);
  });

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
