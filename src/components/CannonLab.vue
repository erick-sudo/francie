<script setup lang="ts">
import * as CANNON from "cannon";
import {
  AxesHelper,
  DirectionalLight,
  DoubleSide,
  Fog,
  HemisphereLight,
  Mesh,
  MeshPhongMaterial,
  PerspectiveCamera,
  RepeatWrapping,
  Scene,
  SphereGeometry,
  TextureLoader,
  Vector3,
  WebGLRenderer,
} from "three";
import {
  OrbitControls,
  ParametricGeometry,
} from "three/examples/jsm/Addons.js";

let camera: PerspectiveCamera, scene: Scene, renderer: WebGLRenderer;
let controls: OrbitControls;

let world: CANNON.World;
let sphereBody: CANNON.Body;
let ballMesh: Mesh;
let particles = [];
let clothGeometry: ParametricGeometry;
let clothMesh: Mesh;

var dt = 1 / 60,
  R = 0.2;

var clothMass = 1; // 1 kg in total
var clothSize = 1; // 1 meter
var Nx = 12;
var Ny = 12;
var mass = (clothMass / Nx) * Ny;

var restDistance = clothSize / Nx;

// Test
var ballSize = 0.1;

var clothFunction = plane(restDistance * Nx, restDistance * Ny);

function plane(width: number, height: number) {
  return function (u: number, v: number) {
    var x = (u - 0.5) * width;
    var y = (v + 0.5) * height;
    var z = 0;
    return new Vector3(x, y, z);
  };
}

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

    const textureLoader = new TextureLoader();
    textureLoader.load("sunflower.jpg", (clothTexture) => {
      console.log("Here");
      clothTexture.wrapS = clothTexture.wrapT = RepeatWrapping;
      clothTexture.anisotropy = 16;

      // cloth material
      const clothMaterial = new MeshPhongMaterial({
        alphaTest: 0.5,
        color: 0xffffff,
        specular: 0x333333,
        emissive: 0x222222,
        //shininess: 5,
        map: clothTexture,
        side: DoubleSide,
      });

      // cloth geometry
      clothGeometry = new ParametricGeometry(clothFunction, Nx, Ny);
      //   clothGeometry.dynamic = true;
      clothGeometry.computeVertexNormals();

      // cloth mesh
      clothMesh = new Mesh(clothGeometry, clothMaterial);
      clothMesh.position.set(0, 0, 0);
      clothMesh.castShadow = true;
      scene.add(clothMesh);
    });

    // Sphere
    const ballGeometry = new SphereGeometry(ballSize, 20, 20);
    const ballMaterial = new MeshPhongMaterial({ color: 0x888888 });
    ballMesh = new Mesh(ballGeometry, ballMaterial);
    ballMesh.castShadow = true;
    scene.add(ballMesh);
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

function initCannon() {
  world = new CANNON.World();
  world.broadphase = new CANNON.NaiveBroadphase();
  world.gravity.set(0, -9.82, 0);
  world.solver.iterations = 20;

  // Materials
  const clothMaterial = new CANNON.Material("cloth_material");
  const sphereMaterial = new CANNON.Material("sphere_material");
  const clothSphereContactMaterial = new CANNON.ContactMaterial(
    clothMaterial,
    sphereMaterial,
    {
      friction: 0.0,
      restitution: 0.0,
    }
  );

  // Adjust constraint equation parameters for ground/ground contact
  clothSphereContactMaterial.contactEquationStiffness = 1e9;
  clothSphereContactMaterial.contactEquationRelaxation = 3;

  // Add contact material to the world
  world.addContactMaterial(clothSphereContactMaterial);

  // Create sphere
  const sphereShape = new CANNON.Sphere(ballSize);
  sphereBody = new CANNON.Body({
    mass: 0,
  });
  sphereBody.addShape(sphereShape);
  sphereBody.position.set(0, 0, 0);
  world.addBody(sphereBody);
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
