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
          <p class="title">{{ content.name }}</p>
          <div class="hr" />
          <p class="description">{{ content.description }}</p>
          <video :ref="videoRef" muted loop>
            <source :src="content.webm" type="video/webm" />
            <source :src="content.mp4" type="video/mp4" />
          </video>
          <button @click="moveToPage(content.route)">ENTER</button>
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

import pofoMp4 from "@/assets/video/pofo-thumbnail.mp4";
import pofoWebm from "@/assets/video/pofo-thumbnail.webm";
import tommyMp4 from "@/assets/video/tommy-thumbnail.mp4";
import tommyWebm from "@/assets/video/tommy-thumbnail.webm";
import lotteriaMp4 from "@/assets/video/lotteria-thumbnail.mp4";
import lotteriaWebm from "@/assets/video/lotteria-thumbnail.webm";
import lawMp4 from "@/assets/video/law-thumbnail.mp4";
import lawWebm from "@/assets/video/law-thumbnail.webm";

const contents = [
  {
    name: "New Portfolio",
    route: "renewalsungpofo.firebaseapp",
    description: "저의 모든 프로젝트를 볼 수 있는 포트폴리오 사이트입니다.",
    webm: pofoWebm,
    mp4: pofoMp4,
  },
  {
    name: "Tommy Future esthetic",
    route: "amazing-prototype.firebaseapp",
    description:
      "다양한 인터랙션 효과를 사용해 Esthetic 프로그램을 소개하는 사이트입니다.",
    webm: tommyWebm,
    mp4: tommyMp4,
  },
  {
    name: "Lotteria Font",
    route: "lotteriafont",
    description:
      "약 10만 명이 방문한 롯데리아의 새로운 폰트 출시를 기념해 제작한 사이트입니다.",
    webm: lotteriaWebm,
    mp4: lotteriaMp4,
  },
  {
    name: "LAW 24",
    route: "law24-prototype.firebaseapp",
    description: "사용자와 변호사를 자동으로 매칭 시켜주는 사이트입니다.",
    webm: lawWebm,
    mp4: lawMp4,
  },
];

const moveToPage = (route) => {
  open(`https://${route}.com`);
};

const canvasRef = ref();
const openBox = ref(0);
const videoArray = ref([]);
const videoRef = (el) => videoArray.value.push(el);
let loadedModel = 0;

let renderer;
let camera;
let raf;
let mixer;
let mixerJellyfish, mixerTurtle;
let fish;
let chest1, chest2, chest3, chest4;

const fishSpeed = 28;
const cameraY = 17; // 카메라 높이
const cameraZ = 14; // 카메라 거리

const scene = new THREE.Scene();
scene.background = new THREE.Color(0x0c6ceb);

// 안개
scene.fog = new THREE.FogExp2(0x00bfff, 0.02);

const params1 = { arc: 0 };
const params2 = { arc: 0 };
const params3 = { arc: 0 };
const torusGeometry = new THREE.TorusGeometry(3, 0.1, 2, 40, params1.arc);
const torusMaterial = new THREE.MeshBasicMaterial({ color: 0x13003f });
const torus1 = new THREE.Mesh(torusGeometry, torusMaterial);
const torus2 = new THREE.Mesh(torusGeometry, torusMaterial);
const torus3 = new THREE.Mesh(torusGeometry, torusMaterial);
torus1.rotation.x = Math.PI * 0.5;
torus1.rotation.z = Math.PI * 0.5;
torus1.position.set(12, 0.5, -55.5);
scene.add(torus1);
torus2.rotation.x = -Math.PI * 0.5;
torus2.position.set(4, 0.5, -55.5);
scene.add(torus2);
torus3.rotation.x = -Math.PI * 0.5;
torus3.position.set(-4, 0.5, -55.5);
scene.add(torus3);

const arcTL1 = gsap.timeline({ paused: true });
arcTL1.to(params1, {
  duration: 4,
  arc: Math.PI * 2,
  ease: "none",
  onUpdate: () => {
    torus1.geometry.dispose();
    torus1.geometry = new THREE.TorusGeometry(3, 0.1, 2, 40, params1.arc);
  },
  onComplete: () => {
    open("https://github.com/swc9803");
  },
});
const arcTL2 = gsap.timeline({ paused: true });
arcTL2.to(params2, {
  duration: 4,
  arc: Math.PI * 2,
  ease: "none",
  onUpdate: () => {
    torus2.geometry.dispose();
    torus2.geometry = new THREE.TorusGeometry(3, 0.1, 2, 40, params2.arc);
  },
  onComplete: () => {
    open("https://codepen.io/swc9803/pens/public");
  },
});
const arcTL3 = gsap.timeline({ paused: true });
arcTL3.to(params3, {
  duration: 4,
  arc: Math.PI * 2,
  ease: "none",
  onUpdate: () => {
    torus3.geometry.dispose();
    torus3.geometry = new THREE.TorusGeometry(3, 0.1, 2, 40, params3.arc);
  },
  onComplete: () => {
    open("mailto:swc9803@gmail.com");
  },
});

// 바닥
const floorGeometry = new THREE.PlaneGeometry(350, 70);
const floorMaterial = new THREE.MeshPhongMaterial({
  color: 0x00bfff,
  side: THREE.DoubleSide,
});
const floor = new THREE.Mesh(floorGeometry, floorMaterial);
floor.receiveShadow = true;
floor.rotation.set(-Math.PI / 2, 0, 0);
floor.position.set(-100, 0, -30);
scene.add(floor);

// light
const loadLight = () => {
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.8);
  const directionalLight = new THREE.DirectionalLight(0xf8f8ff, 4);
  directionalLight.castShadow = true;
  directionalLight.position.set(0, 150, -85);
  directionalLight.target.position.set(-100, 0, -35);

  directionalLight.shadow.camera.left = -140;
  directionalLight.shadow.camera.right = 140;
  directionalLight.shadow.camera.top = 140;
  directionalLight.shadow.camera.bottom = -140;
  directionalLight.shadow.mapSize.width = 2048;
  directionalLight.shadow.mapSize.height = 2048;

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

      chest.scene.scale.set(0.03, 0.03, 0.03);
      chest.scene.rotation.y = Math.PI;
      chest.scene.position.set(x, 0, z);
      chest.scene.traverse((node) => {
        if (node.isMesh) {
          node.castShadow = true;
        }
      });
      scene.add(chest.scene);

      loadedModel++;

      resolve(chest);
    });
  });
};
// fish
const loadFish = () => {
  gltfLoader.load("/fish.glb", (gltf) => {
    fish = gltf.scene;
    fish.rotation.set(0, Math.PI, 0);
    fish.position.y = 2.5;
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.castShadow = true;
        node.receiveShadow = true;
      }
    });
    scene.add(fish);

    loadedModel++;

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
    gltf.scene.rotation.y = Math.PI * 0.975;
    gltf.scene.position.set(-15, 0, -20);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.castShadow = true;
      }
    });
    scene.add(gltf.scene);

    loadedModel++;
  });
  gltfLoader.load("/sign-about.glb", (gltf) => {
    gltf.scene.rotation.y = Math.PI * 0.4;
    gltf.scene.position.set(10, 0, -30);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.castShadow = true;
      }
    });
    scene.add(gltf.scene);

    loadedModel++;
  });
  gltfLoader.load("/sign-playground.glb", (gltf) => {
    gltf.scene.rotation.y = Math.PI * 0.025;
    gltf.scene.position.set(15, 0, -20);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.castShadow = true;
      }
    });
    scene.add(gltf.scene);

    loadedModel++;
  });
};
// projectImage
const loadFrame = () => {
  // portfolio
  gltfLoader.load("/projects/frame-pofo1.glb", (gltf) => {
    gltf.scene.rotation.y = Math.PI * 1.1;
    gltf.scene.position.set(-45, 0, -20);
    gltf.scene.scale.set(1.1, 1.1, 1.1);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.castShadow = true;
      }
    });
    scene.add(gltf.scene);

    loadedModel++;
  });
  gltfLoader.load("/projects/frame-pofo2.glb", (gltf) => {
    gltf.scene.rotation.y = Math.PI;
    gltf.scene.position.set(-58, 0, -17.5);
    gltf.scene.scale.set(1.1, 1.1, 1.1);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.castShadow = true;
      }
    });
    scene.add(gltf.scene);

    loadedModel++;
  });
  gltfLoader.load("/projects/frame-pofo3.glb", (gltf) => {
    gltf.scene.rotation.y = Math.PI * 0.9;
    gltf.scene.position.set(-71, 0, -20);
    gltf.scene.scale.set(1.1, 1.1, 1.1);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.castShadow = true;
      }
    });
    scene.add(gltf.scene);

    loadedModel++;
  });
  // tommy
  gltfLoader.load("/projects/frame-tommy1.glb", (gltf) => {
    gltf.scene.rotation.y = Math.PI * 1.1;
    gltf.scene.position.set(-100, 0, -20);
    gltf.scene.scale.set(1.1, 1.1, 1.1);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.castShadow = true;
      }
    });
    scene.add(gltf.scene);

    loadedModel++;
  });
  gltfLoader.load("/projects/frame-tommy2.glb", (gltf) => {
    gltf.scene.rotation.y = Math.PI;
    gltf.scene.position.set(-113, 0, -17.5);
    gltf.scene.scale.set(1.1, 1.1, 1.1);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.castShadow = true;
      }
    });
    scene.add(gltf.scene);

    loadedModel++;
  });
  gltfLoader.load("/projects/frame-tommy3.glb", (gltf) => {
    gltf.scene.rotation.y = Math.PI * 0.9;
    gltf.scene.position.set(-126, 0, -20);
    gltf.scene.scale.set(1.1, 1.1, 1.1);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.castShadow = true;
      }
    });
    scene.add(gltf.scene);

    loadedModel++;
  });
  // lotteria
  gltfLoader.load("/projects/frame-lotteria1.glb", (gltf) => {
    gltf.scene.rotation.y = Math.PI * 1.1;
    gltf.scene.position.set(-155, 0, -20);
    gltf.scene.scale.set(1.1, 1.1, 1.1);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.castShadow = true;
      }
    });
    scene.add(gltf.scene);

    loadedModel++;
  });
  gltfLoader.load("/projects/frame-lotteria2.glb", (gltf) => {
    gltf.scene.rotation.y = Math.PI;
    gltf.scene.position.set(-168, 0, -17.5);
    gltf.scene.scale.set(1.1, 1.1, 1.1);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.castShadow = true;
      }
    });
    scene.add(gltf.scene);

    loadedModel++;
  });
  gltfLoader.load("/projects/frame-lotteria3.glb", (gltf) => {
    gltf.scene.rotation.y = Math.PI * 0.9;
    gltf.scene.position.set(-181, 0, -20);
    gltf.scene.scale.set(1.1, 1.1, 1.1);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.castShadow = true;
      }
    });
    scene.add(gltf.scene);

    loadedModel++;
  });
  // law24
  gltfLoader.load("/projects/frame-law1.glb", (gltf) => {
    gltf.scene.rotation.y = Math.PI * 1.1;
    gltf.scene.position.set(-210, 0, -20);
    gltf.scene.scale.set(1.1, 1.1, 1.1);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.castShadow = true;
      }
    });
    scene.add(gltf.scene);

    loadedModel++;
  });
  gltfLoader.load("/projects/frame-law2.glb", (gltf) => {
    gltf.scene.rotation.y = Math.PI;
    gltf.scene.position.set(-223, 0, -17.5);
    gltf.scene.scale.set(1.1, 1.1, 1.1);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.castShadow = true;
      }
    });
    scene.add(gltf.scene);

    loadedModel++;
  });
  gltfLoader.load("/projects/frame-law3.glb", (gltf) => {
    gltf.scene.rotation.y = Math.PI * 0.9;
    gltf.scene.position.set(-236, 0, -20);
    gltf.scene.scale.set(1.1, 1.1, 1.1);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.castShadow = true;
      }
    });
    scene.add(gltf.scene);

    loadedModel++;
  });
};
const loadLogo = () => {
  gltfLoader.load("/github.glb", (gltf) => {
    gltf.scene.rotation.y = Math.PI;
    gltf.scene.position.set(12, 0.5, -55);
    gltf.scene.scale.set(3, 3, 3);
    scene.add(gltf.scene);

    loadedModel++;
  });
  gltfLoader.load("/codepen.glb", (gltf) => {
    gltf.scene.rotation.y = Math.PI;
    gltf.scene.position.set(4, 0.5, -55);
    gltf.scene.scale.set(3, 3, 3);
    scene.add(gltf.scene);

    loadedModel++;
  });
  gltfLoader.load("/email.glb", (gltf) => {
    gltf.scene.rotation.y = Math.PI;
    gltf.scene.position.set(-4, 0.5, -55);
    gltf.scene.scale.set(3, 3, 3);
    scene.add(gltf.scene);

    loadedModel++;
  });
};

const loadDecoration = () => {
  // 공사 중
  gltfLoader.load("/construction.glb", (gltf) => {
    gltf.scene.rotation.y = Math.PI * 1.05;
    gltf.scene.position.set(60, 0, -23);
    gltf.scene.scale.set(7, 7, 7);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.castShadow = true;
      }
    });
    scene.add(gltf.scene);

    loadedModel++;
  });

  // 해파리
  gltfLoader.load("/decoration/jellyfish.glb", (gltf) => {
    mixerJellyfish = new THREE.AnimationMixer(gltf.scene);
    const action = mixerJellyfish.clipAction(gltf.animations[0]);
    action.play();

    gltf.scene.rotation.x = -Math.PI * 1.5;
    gltf.scene.rotation.z = Math.PI * 1.5;
    gltf.scene.position.set(0, 15, -30);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.castShadow = true;
      }
    });
    scene.add(gltf.scene);

    gsap.to(gltf.scene.position, {
      x: 10,
      yoyo: true,
      repeat: -1,
      duration: 12,
      ease: "power1.out",
    });

    loadedModel++;
  });

  // 거북
  gltfLoader.load("/decoration/turtle.glb", (gltf) => {
    mixerTurtle = new THREE.AnimationMixer(gltf.scene);
    const action = mixerTurtle.clipAction(gltf.animations[0]);
    action.play();

    gltf.scene.scale.set(3, 3, 3);
    gltf.scene.rotation.y = Math.PI * 0.5;
    gltf.scene.position.set(5, 10, -20);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.castShadow = true;
      }
    });
    scene.add(gltf.scene);

    gsap.to(gltf.scene.position, {
      x: 10,
      yoyo: true,
      repeat: -1,
      duration: 12,
      ease: "power1.out",
    });

    loadedModel++;
  });

  // 조개1
  gltfLoader.load("/decoration/clam1.glb", (gltf) => {
    gltf.scene.rotation.x = -Math.PI * 0.5;
    gltf.scene.rotation.z = -Math.PI * 0.1;
    gltf.scene.scale.set(0.03, 0.03, 0.03);
    gltf.scene.position.set(0, 0.3, 0);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.receiveShadow = true;
      }
    });
    scene.add(gltf.scene);

    loadedModel++;
  });

  // 산호
  gltfLoader.load("/decoration/coral1.glb", (gltf) => {
    gltf.scene.scale.set(0.5, 0.5, 0.5);
    gltf.scene.position.set(-5, 0.3, -5);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.castShadow = true;
      }
    });
    scene.add(gltf.scene);

    loadedModel++;
  });
  gltfLoader.load("/decoration/coral2.glb", (gltf) => {
    gltf.scene.rotation.y = Math.PI * 2;
    gltf.scene.scale.set(2, 2, 2);
    gltf.scene.position.set(5, 0.3, -5);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.castShadow = true;
      }
    });
    scene.add(gltf.scene);

    loadedModel++;
  });

  // 불가사리
  gltfLoader.load("/decoration/seastar.glb", (gltf) => {
    gltf.scene.rotation.x = -Math.PI * 0.5;
    gltf.scene.scale.set(35, 35, 35);
    gltf.scene.position.set(5, 0.3, -10);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.receiveShadow = true;
      }
    });
    scene.add(gltf.scene);

    loadedModel++;
  });

  // 해초
  gltfLoader.load("/decoration/seaweed1.glb", (gltf) => {
    gltf.scene.scale.set(15, 15, 15);
    gltf.scene.position.set(-5, 0.3, -10);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.castShadow = true;
      }
    });
    scene.add(gltf.scene);

    loadedModel++;
  });
  gltfLoader.load("/decoration/seaweed2.glb", (gltf) => {
    gltf.scene.scale.set(10, 10, 10);
    gltf.scene.position.set(-5, 5, -15);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.castShadow = true;
      }
    });
    scene.add(gltf.scene);

    loadedModel++;
  });

  // 소라 껍데기
  gltfLoader.load("/decoration/shell1.glb", (gltf) => {
    gltf.scene.rotation.z = -Math.PI * 0.7;
    gltf.scene.scale.set(50, 50, 50);
    gltf.scene.position.set(5, 1.3, -15);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.castShadow = true;
      }
    });
    scene.add(gltf.scene);

    loadedModel++;
  });
  gltfLoader.load("/decoration/shell2.glb", (gltf) => {
    gltf.scene.scale.set(0.5, 0.5, 0.5);
    gltf.scene.position.set(5, 0, -20);
    gltf.scene.traverse((node) => {
      if (node.isMesh) {
        node.castShadow = true;
      }
    });
    scene.add(gltf.scene);

    loadedModel++;
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
  if (loadedModel >= 21) {
    isClicked.value = true;
    mouse.x = (e.clientX / canvasRef.value.offsetWidth) * 2 - 1;
    mouse.y = -(e.clientY / canvasRef.value.offsetHeight) * 2 + 1;
  }
};
const onMouseMove = (e) => {
  if (loadedModel >= 21) {
    mouse.x = (e.clientX / canvasRef.value.offsetWidth) * 2 - 1;
    mouse.y = -(e.clientY / canvasRef.value.offsetHeight) * 2 + 1;
  }
};
const onMouseUp = () => {
  if (loadedModel >= 21) {
    isClicked.value = false;
  }
};
const onTouchStart = (e) => {
  if (loadedModel >= 21) {
    isClicked.value = true;
    mouse.x = (e.touches[0].clientX / canvasRef.value.offsetWidth) * 2 - 1;
    mouse.y = -(e.touches[0].clientY / canvasRef.value.offsetHeight) * 2 + 1;
  }
};
const onTouchMove = (e) => {
  if (loadedModel >= 21) {
    mouse.x = (e.touches[0].clientX / canvasRef.value.offsetWidth) * 2 - 1;
    mouse.y = -(e.touches[0].clientY / canvasRef.value.offsetHeight) * 2 + 1;
  }
};
const onTouchEnd = () => {
  if (loadedModel >= 21) {
    isClicked.value = false;
  }
};

const offset = new THREE.Vector3(0, cameraY, -cameraZ);
const clock = new THREE.Clock();
const animate = () => {
  const deltaTime = clock.getDelta();

  camera.updateMatrixWorld();
  renderer.render(scene, camera);
  raf = requestAnimationFrame(animate);

  if (mixer) mixer.update(deltaTime);
  if (mixerJellyfish) mixerJellyfish.update(deltaTime);
  if (mixerTurtle) mixerTurtle.update(deltaTime);

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
            videoArray.value[0].play();
            box1AnimationPlayed = true;
          } else if (box1distance > 5 && box1AnimationPlayed) {
            mixer = new THREE.AnimationMixer(chest1.scene);
            const action = mixer.clipAction(chest1.animations[3]);
            action.setLoop(THREE.LoopOnce);
            action.clampWhenFinished = true;
            action.play();
            openBox.value = 0;
            videoArray.value[0].pause();
            box1AnimationPlayed = false;
          }

          if (box2distance <= 5 && !box2AnimationPlayed) {
            mixer = new THREE.AnimationMixer(chest2.scene);
            const action = mixer.clipAction(chest2.animations[1]);
            action.setLoop(THREE.LoopOnce);
            action.clampWhenFinished = true;
            action.play();
            openBox.value = 2;
            videoArray.value[1].play();
            box2AnimationPlayed = true;
          } else if (box2distance > 5 && box2AnimationPlayed) {
            mixer = new THREE.AnimationMixer(chest2.scene);
            const action = mixer.clipAction(chest2.animations[3]);
            action.setLoop(THREE.LoopOnce);
            action.clampWhenFinished = true;
            action.play();
            openBox.value = 0;
            videoArray.value[1].pause();
            box2AnimationPlayed = false;
          }

          if (box3distance <= 5 && !box3AnimationPlayed) {
            mixer = new THREE.AnimationMixer(chest3.scene);
            const action = mixer.clipAction(chest3.animations[1]);
            action.setLoop(THREE.LoopOnce);
            action.clampWhenFinished = true;
            action.play();
            openBox.value = 3;
            videoArray.value[2].play();
            box3AnimationPlayed = true;
          } else if (box3distance > 5 && box3AnimationPlayed) {
            mixer = new THREE.AnimationMixer(chest3.scene);
            const action = mixer.clipAction(chest3.animations[3]);
            action.setLoop(THREE.LoopOnce);
            action.clampWhenFinished = true;
            action.play();
            openBox.value = 0;
            videoArray.value[2].pause();
            box3AnimationPlayed = false;
          }

          if (box4distance <= 5 && !box4AnimationPlayed) {
            mixer = new THREE.AnimationMixer(chest4.scene);
            const action = mixer.clipAction(chest4.animations[1]);
            action.setLoop(THREE.LoopOnce);
            action.clampWhenFinished = true;
            action.play();
            openBox.value = 4;
            videoArray.value[3].play();
            box4AnimationPlayed = true;
          } else if (box4distance > 5 && box4AnimationPlayed) {
            mixer = new THREE.AnimationMixer(chest4.scene);
            const action = mixer.clipAction(chest4.animations[3]);
            action.setLoop(THREE.LoopOnce);
            action.clampWhenFinished = true;
            action.play();
            openBox.value = 0;
            videoArray.value[3].pause();
            box4AnimationPlayed = false;
          }

          const info1distance = fish.position.distanceTo(torus1.position);
          const info2distance = fish.position.distanceTo(torus2.position);
          const info3distance = fish.position.distanceTo(torus3.position);
          if (info1distance <= 3) {
            arcTL1.play();
          } else if (info1distance > 3) {
            arcTL1.reverse();
          }
          if (info2distance <= 3) {
            arcTL2.play();
          } else if (info2distance > 3) {
            arcTL2.reverse();
          }
          if (info3distance <= 3) {
            arcTL3.play();
          } else if (info3distance > 3) {
            arcTL3.reverse();
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
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.PCFSoftShadowMap;

  [chest1, chest2, chest3, chest4] = await Promise.all([
    loadChest(-58, -27),
    loadChest(-113, -27),
    loadChest(-168, -27),
    loadChest(-223, -27),
  ]);
  loadLight();
  loadFish();
  loadSign();
  loadFrame();
  loadLogo();
  loadDecoration();

  camera = new THREE.PerspectiveCamera(
    75,
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
  .message {
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate3d(-50%, -50%, 0);
    width: 60%;
    pointer-events: none;
    z-index: 3;
    @media (max-width: 768px) {
      width: 95%;
    }
    .content-wrapper {
      display: flex;
      flex-direction: column;
      align-items: center;
      text-align: center;
      padding: 20px;
      border-radius: 1em;
      transform-origin: center;
      background: rgb(46, 43, 41);
      opacity: 0.8;
      @media (max-width: 768px) {
        padding: 40px 20px;
      }
      .title {
        margin-bottom: 10px;
        color: white;
        font-size: 2em;
        font-weight: 900;
      }
      .hr {
        margin-bottom: 30px;
        border-radius: 50%;
        width: 140px;
        height: 2px;
        background: #ffffff;
        @media (max-width: 768px) {
          margin-bottom: 40px;
        }
      }
      .description {
        margin-bottom: 20px;
        color: #ffffff;
        font-size: 1.25em;
        font-weight: 700;
        line-height: 1.4em;
        word-break: keep-all;
        @media (max-width: 768px) {
          margin-bottom: 60px;
        }
      }
      video {
        margin-bottom: 40px;
        width: 50%;
        object-fit: cover;
        @media (max-width: 768px) {
          margin-bottom: 100px;
          width: 100%;
        }
      }
      button {
        border-radius: 16px;
        width: 180px;
        height: 60px;
        color: #ffffff;
        border: 1px solid #ffffff;
        background: linear-gradient(
          270deg,
          rgba(255, 255, 255, 1),
          rgba(255, 255, 255, 1),
          rgba(209, 209, 209, 0.8),
          rgba(102, 102, 102, 0),
          rgba(0, 0, 0, 0)
        );
        background-position: 0 50%;
        background-size: 350% 300%;
        transition: all 0.5s ease-out;
        pointer-events: auto;
        font-size: 1.25em;
        font-weight: 700;
        &:hover {
          color: black;
          background-position: 100% 50%;
        }
      }
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
