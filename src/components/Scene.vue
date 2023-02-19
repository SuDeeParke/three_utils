<template>
  <div class="container" ref="container"></div>
</template>

<script setup>
import * as THREE from 'three'
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { RoomEnvironment } from 'three/examples/jsm/environments/RoomEnvironment'
import Stats from 'three/examples/jsm/libs/stats.module'
const container = ref(null);

onMounted(() => {
  const scene = new THREE.Scene()
  const axesHelper = new THREE.AxesHelper(100)
  scene.add(axesHelper);

  const spotLight = new THREE.SpotLight(0xffffff);
  spotLight.position.set(100, 300, 100);

  spotLight.castShadow = true;

  spotLight.shadow.mapSize.width = 1024;
  spotLight.shadow.mapSize.height = 1024;

  spotLight.shadow.camera.near = 1;
  spotLight.shadow.camera.far = 500;
  spotLight.shadow.camera.fov = 60;

  scene.add(spotLight);

  const objLoader = new THREE.ObjectLoader();
 
  objLoader.load('/public/model/scene.json', 
    // onLoad
    (obj) => {
      obj.traverse((o) => {
        if(o instanceof THREE.Mesh){
          o.castShadow = true;
          o.receiveShadow = true; 
        }
      })
      scene.add(obj);
    },
    // onProcess
    (process) => {
      // console.log(process)
    },
    // onError
    (err)=> {
      console.log(err)
    }
  );

  const camera = new THREE.PerspectiveCamera(60,window.innerWidth / window.innerHeight, 1, 500);
  camera.position.x = 5;
  camera.position.y = 5;
  camera.position.z = 20;

  const renderer = new THREE.WebGLRenderer({
    antialias: true,
    powerPreference: 'high-performance',
    precision: 'highp',// 着色器精度
  });
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.PCFSoftShadowMap; // default THREE.PCFShadowMap
  renderer.setSize(window.innerWidth, window.innerHeight);
  container.value.appendChild(renderer.domElement);

  const pmremGenerator = new THREE.PMREMGenerator(renderer)
  pmremGenerator.compileEquirectangularShader()
  // environment
  scene.environment = pmremGenerator.fromScene(new RoomEnvironment(), 0.04).texture
  scene.background = new THREE.Color(0x333333)
  scene.translateY(-10)  

  const controls = new OrbitControls(camera, renderer.domElement)
  // 平行移动
  controls.enablePan = true
  // 惯性
  controls.enableDamping = true
  // 旋转
  controls.autoRotate = false
  controls.autoRotateSpeed = 0.6
  controls.dampingFactor = 0.1
  controls.minDistance = 1
  controls.maxDistance = 100

  const panelWidth =  82
  const stats_FPS = new Stats();
  stats_FPS.showPanel(0);
  stats_FPS.domElement.style.left = `${panelWidth*0}px`;
  document.body.appendChild(stats_FPS.domElement);

  const stats_MS = new Stats();
  stats_MS.showPanel(1);
  stats_MS.domElement.style.left = `${panelWidth * 1}px`;

  document.body.appendChild(stats_MS.domElement);
  const stats_MB = new Stats();
  stats_MB.showPanel(2);
  stats_MB.domElement.style.left = `${panelWidth * 2}px`;
  document.body.appendChild(stats_MB.domElement);

  const stats_update = () => {
    stats_FPS.update();
    stats_MS.update();
    stats_MB.update();
  }
  renderer.setAnimationLoop(() => {
    stats_update()
    controls.update()
    renderer.render(scene, camera)
  })
})
</script>

<style scoped>
.container{
  position: relative;
  display: block;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 1);
}
</style>