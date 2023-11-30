<template>
  <div class="container">
    <div
      ref="canvasRef"
      class="wrapper"
      @click="onClick"
      @mousedown="onMouseDown"
      @mousemove="onMouseMove"
      @mouseup="onMouseUp"
      @touchstart="onTouchStart"
      @touchmove="onTouchMove"
      @touchend="onTouchEnd"
    />
    <div v-for="(content, index) in contents" :key="content" class="message">
      <transition name="fade">
        <div v-show="openBox === index + 1" class="content-wrapper">
          <p>{{ content.name }}</p>
          <button @click="moveToPage(content.route)">보러가기</button>
        </div>
      </transition>
    </div>
  </div>
</template>

<script setup>
import gsap from "gsap";
import * as THREE from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader.js";

const contents = [
  { name: "포트폴리오", route: "newsungpf.firebaseapp.com" },
  { name: "갤러리", route: "sung-gallery.firebaseapp.com" },
  { name: "깃허브", route: "github.com/swc9803" },
  { name: "코드팬", route: "codepen.io/swc9803" },
];

const moveToPage = (route) => {
  open(`https://${route}`);
};

const canvasRef = ref();
const openBox = ref(0);

let renderer;
let camera;
let raf;
let mixer;
let fish;
let chest1, chest2, chest3, chest4;

const fishSpeed = 8;
const cameraY = 15; // obj로부터의 카메라 높이
const cameraZ = 13; // obj로부터의 카메라 거리

const scene = new THREE.Scene();
scene.background = new THREE.Color(0x0c6ceb);

// 안개
scene.fog = new THREE.FogExp2(0x00bfff, 0.02); // 밀도

// 바닥
const floorGeometry = new THREE.PlaneGeometry(200, 100);
const floorMaterial = new THREE.MeshBasicMaterial({ color: 0x00bfff });
const floor = new THREE.Mesh(floorGeometry, floorMaterial);
floor.rotation.set(-Math.PI / 2, 0, 0);
floor.position.set(0, -1, -35);
scene.add(floor);

// light
const loadLight = () => {
  const ambientLight = new THREE.AmbientLight(0x8ae2ff, 0.4);
  const directionalLight = new THREE.DirectionalLight(0x8ae2ff, 5);
  directionalLight.position.set(15, 30, -20);
  directionalLight.target.position.set(-5, 0, -5);
  scene.add(ambientLight);
  scene.add(directionalLight);
  scene.add(directionalLight.target);
};

const gltfLoader = new GLTFLoader();
const dracoLoader = new DRACOLoader();
gltfLoader.setDRACOLoader(dracoLoader);
dracoLoader.setDecoderPath(
  "https://raw.githubusercontent.com/mrdoob/three.js/dev/examples/js/libs/draco/",
);
// chest
const loadChest = (x, z) => {
  return new Promise((resolve) => {
    gltfLoader.load("/chest.glb", (gltf) => {
      const chest = gltf;
      if (gltf.animations && gltf.animations.length > 0) {
        chest.mixer = new THREE.AnimationMixer(chest.scene);
        gltf.animations.forEach((clip) => {
          chest.mixer.clipAction(clip).play();
        });
      }

      chest.scene.scale.set(0.02, 0.02, 0.02);
      chest.scene.rotation.y = Math.PI;
      chest.scene.position.set(x, 0, z);
      scene.add(chest.scene);
      resolve(chest);
    });
  });
};
// fish
const loadFish = () => {
  gltfLoader.load("/fish.glb", (gltf) => {
    fish = gltf.scene;
    fish.rotation.set(0, Math.PI, 0);
    scene.add(fish);
    // mixer = new THREE.AnimationMixer(gltf.scene);
    // const action = mixer.clipAction(gltf.animations[0]);
    // console.log(mixer.clipAction(gltf.animations));
    // action.setLoop(THREE.LoopOnce);
    // action.clampWhenFinished = true;
    // action.play();
  });
};
// sign
const loadSign = () => {
  gltfLoader.load("/sign-projects.glb", (gltf) => {
    gltf.scene.rotation.y = Math.PI;
    gltf.scene.position.set(-10, 0, -20);
    scene.add(gltf.scene);
  });
  gltfLoader.load("/sign-about.glb", (gltf) => {
    gltf.scene.rotation.y = Math.PI / 2.5;
    gltf.scene.position.set(10, 0, -30);
    scene.add(gltf.scene);
  });
};
// projectImage
const loadFrame = () => {
  gltfLoader.load("/frame.glb", (gltf) => {
    gltf.scene.rotation.y = Math.PI;
    gltf.scene.position.set(0, 0, 0);
    scene.add(gltf.scene);
  });
};

const onResize = () => {
  const vh = window.innerHeight * 0.01;
  document.documentElement.style.setProperty("--vh", `${vh}px`);

  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.setSize(canvasRef.value.offsetWidth, canvasRef.value.offsetHeight);
  canvasRef.value.appendChild(renderer.domElement);
  camera.aspect = canvasRef.value.offsetWidth / canvasRef.value.offsetHeight;
  camera.updateProjectionMatrix();
};

let box1AnimationPlayed = false;
let box2AnimationPlayed = false;
let box3AnimationPlayed = false;
let box4AnimationPlayed = false;

const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();

const isClicked = ref(false);
const onMouseDown = (e) => {
  isClicked.value = true;
  mouse.x = (e.clientX / canvasRef.value.offsetWidth) * 2 - 1;
  mouse.y = -(e.clientY / canvasRef.value.offsetHeight) * 2 + 1;
};
const onMouseMove = (e) => {
  mouse.x = (e.clientX / canvasRef.value.offsetWidth) * 2 - 1;
  mouse.y = -(e.clientY / canvasRef.value.offsetHeight) * 2 + 1;
};
const onMouseUp = () => {
  isClicked.value = false;
};
const onTouchStart = (e) => {
  isClicked.value = true;
  mouse.x = (e.touches[0].clientX / canvasRef.value.offsetWidth) * 2 - 1;
  mouse.y = -(e.touches[0].clientY / canvasRef.value.offsetHeight) * 2 + 1;
};
const onTouchMove = (e) => {
  mouse.x = (e.touches[0].clientX / canvasRef.value.offsetWidth) * 2 - 1;
  mouse.y = -(e.touches[0].clientY / canvasRef.value.offsetHeight) * 2 + 1;
};
const onTouchEnd = () => {
  isClicked.value = false;
};

const offset = new THREE.Vector3(0, cameraY, -cameraZ);
const clock = new THREE.Clock();
const animate = () => {
  const deltaTime = clock.getDelta();

  camera.updateMatrixWorld();
  renderer.render(scene, camera);
  raf = requestAnimationFrame(animate);

  if (mixer) mixer.update(deltaTime);

  if (fish) {
    const targetPosition = fish.position.clone().add(offset);
    // targetPosition.x -= 2;
    camera.position.copy(targetPosition);
    camera.lookAt(fish.position);
  }

  if (isClicked.value) {
    raycaster.setFromCamera(mouse, camera);
    const intersects = raycaster.intersectObject(floor);

    if (intersects.length > 0) {
      const intersectionPoint = intersects[0].point;
      console.log("클릭 좌표:", intersectionPoint);

      const distance = fish.position.distanceTo(intersectionPoint);
      const duration = distance / fishSpeed;

      // fish 속도에 따라 애니메이션 속도 증가 및 감소, 움직이는 동안 무한 반복

      gsap.killTweensOf(fish.position);
      gsap.to(fish.position, {
        x: intersectionPoint.x,
        z: intersectionPoint.z,
        duration,
        onUpdate: () => {
          const box1distance = fish.position.distanceTo(chest1.scene.position);
          const box2distance = fish.position.distanceTo(chest2.scene.position);
          const box3distance = fish.position.distanceTo(chest3.scene.position);
          const box4distance = fish.position.distanceTo(chest4.scene.position);

          if (box1distance <= 5 && !box1AnimationPlayed) {
            mixer = new THREE.AnimationMixer(chest1.scene);
            const action = mixer.clipAction(chest1.animations[1]);
            action.setLoop(THREE.LoopOnce);
            action.clampWhenFinished = true;
            action.play();
            openBox.value = 1;
            box1AnimationPlayed = true;
          } else if (box1distance > 5 && box1AnimationPlayed) {
            mixer = new THREE.AnimationMixer(chest1.scene);
            const action = mixer.clipAction(chest1.animations[3]);
            action.setLoop(THREE.LoopOnce);
            action.clampWhenFinished = true;
            action.play();
            openBox.value = 0;
            box1AnimationPlayed = false;
          }

          if (box2distance <= 5 && !box2AnimationPlayed) {
            mixer = new THREE.AnimationMixer(chest2.scene);
            const action = mixer.clipAction(chest2.animations[1]);
            action.setLoop(THREE.LoopOnce);
            action.clampWhenFinished = true;
            action.play();
            openBox.value = 2;
            box2AnimationPlayed = true;
          } else if (box2distance > 5 && box2AnimationPlayed) {
            mixer = new THREE.AnimationMixer(chest2.scene);
            const action = mixer.clipAction(chest2.animations[3]);
            action.setLoop(THREE.LoopOnce);
            action.clampWhenFinished = true;
            action.play();
            openBox.value = 0;
            box2AnimationPlayed = false;
          }

          if (box3distance <= 5 && !box3AnimationPlayed) {
            mixer = new THREE.AnimationMixer(chest3.scene);
            const action = mixer.clipAction(chest3.animations[1]);
            action.setLoop(THREE.LoopOnce);
            action.clampWhenFinished = true;
            action.play();
            openBox.value = 3;
            box3AnimationPlayed = true;
          } else if (box3distance > 5 && box3AnimationPlayed) {
            mixer = new THREE.AnimationMixer(chest3.scene);
            const action = mixer.clipAction(chest3.animations[3]);
            action.setLoop(THREE.LoopOnce);
            action.clampWhenFinished = true;
            action.play();
            openBox.value = 0;
            box3AnimationPlayed = false;
          }

          if (box4distance <= 5 && !box4AnimationPlayed) {
            mixer = new THREE.AnimationMixer(chest4.scene);
            const action = mixer.clipAction(chest4.animations[1]);
            action.setLoop(THREE.LoopOnce);
            action.clampWhenFinished = true;
            action.play();
            openBox.value = 4;
            box4AnimationPlayed = true;
          } else if (box4distance > 5 && box4AnimationPlayed) {
            mixer = new THREE.AnimationMixer(chest4.scene);
            const action = mixer.clipAction(chest4.animations[3]);
            action.setLoop(THREE.LoopOnce);
            action.clampWhenFinished = true;
            action.play();
            openBox.value = 0;
            box4AnimationPlayed = false;
          }
        },
      });

      const lookAtPoint = new THREE.Vector3(
        intersectionPoint.x,
        fish.position.y,
        intersectionPoint.z,
      );
      fish.lookAt(lookAtPoint);
    }
  }
};

onMounted(async () => {
  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });

  [chest1, chest2, chest3, chest4] = await Promise.all([
    loadChest(15, 15),
    loadChest(-15, 15),
    loadChest(15, -15),
    loadChest(-15, -15),
  ]);
  loadLight();
  loadFish();
  loadSign();
  loadFrame();

  camera = new THREE.PerspectiveCamera(
    80,
    canvasRef.value.offsetWidth / canvasRef.value.offsetHeight,
    0.1,
    1000,
  );

  onResize();
  animate();

  window.addEventListener("resize", onResize);
});

onBeforeUnmount(() => {
  cancelAnimationFrame(raf);
  renderer.dispose();

  window.removeEventListener("resize", onResize);
});
</script>

<style lang="scss" scoped>
.container {
  position: fixed;
  width: 100%;
  height: calc(var(--vh) * 100);
  .wrapper {
    width: 100%;
    height: 100%;
  }
}
.message {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 60%;
  height: 60%;
  pointer-events: none;
  z-index: 3;
  .content-wrapper {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    width: 100%;
    height: 100%;
    padding: 20px;
    transform-origin: center;
    text-align: center;
    font-size: 2em;
    color: white;
    background: rgb(175, 175, 175);
    opacity: 0.8;
    button {
      border-radius: 16px;
      background: white;
      pointer-events: auto;
    }
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: 0.7s ease-in-out;
}
.fade-enter-from,
.fade-leave-to {
  // opacity: 0;
  transform: scaleY(0);
}
</style>
