<template>
  <div ref="screenDom" class="w-full h-full"></div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { RGBELoader } from 'three/addons/loaders/RGBELoader.js';

const screenDom = ref<HTMLDivElement | null>(null);
const scene = new THREE.Scene();

onMounted(() => {
  init();
});
const init = () => {
  if (!screenDom.value) return;
  // 透视摄像机
  const camera = new THREE.PerspectiveCamera(45, screenDom.value.clientWidth / screenDom.value.clientHeight, 0.25, 20);
  camera.position.set(-1.8, 0.6, 2.7);

  new RGBELoader().setPath('/textures/').load('royal_esplanade_1k.hdr', (texture: { mapping: any }) => {
    texture.mapping = THREE.EquirectangularReflectionMapping;
    scene.background = texture;
    scene.environment = texture;
    renderer.render(scene, camera);

    // model
    const loader = new GLTFLoader().setPath('/glTF/');
    loader.load('DamagedHelmet.gltf', (gltf: { scene: any }) => {
      scene.add(gltf.scene);
      renderer.render(scene, camera);
    });
  });

  const renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.setSize(screenDom.value.clientWidth, screenDom.value.clientHeight);
  renderer.toneMapping = THREE.ACESFilmicToneMapping;
  renderer.toneMappingExposure = 1;
  screenDom.value.appendChild(renderer.domElement);

  const controls = new OrbitControls(camera, renderer.domElement);
  controls.addEventListener('change', () => {
    renderer.render(scene, camera);
  });
  controls.minDistance = 2;
  controls.maxDistance = 10;
  controls.target.set(0, 0, -0.2);
  controls.update();
};
</script>
