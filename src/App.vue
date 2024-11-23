<template>
  <div ref="container" id="model-fox-desktop-en" style="width: 1000px; height: 1000px" />
</template>

<script setup lang="js">
  import * as THREE from "three";
  import {GLTFLoader} from "three/examples/jsm/loaders/GLTFLoader";
  import {MeshoptDecoder} from "three/examples/jsm/libs/meshopt_decoder.module.js";
  import {onMounted, ref} from "vue";

  const container = ref(null);

  function loadFoxDesktopModel(containerElement, modelPath) {
    const scene = new THREE.Scene();

    // Camera
    const camera = new THREE.PerspectiveCamera(
        100,
        containerElement.clientWidth / containerElement.clientHeight,
        0.1,
        1000
    );

    // Renderer
    const renderer = new THREE.WebGLRenderer({alpha: true});
    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.setSize(containerElement.clientWidth, containerElement.clientWidth / (16 / 9));

    // Append renderer to the container element
    containerElement.appendChild(renderer.domElement);

    // Lights
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.4);
    scene.add(ambientLight);

    const directionalLight1 = new THREE.DirectionalLight(0xffffff, 6);
    directionalLight1.position.set(2, 2, 5).normalize();
    scene.add(directionalLight1);

    const directionalLight2 = new THREE.DirectionalLight(0xffffff, 1);
    directionalLight2.position.set(-2, 1, 3).normalize();
    scene.add(directionalLight2);

    renderer.setClearColor(0x000000, 0);

    // GLTF Loader
    const loader = new GLTFLoader();
    loader.setMeshoptDecoder(MeshoptDecoder);

    let mixer;

    loader.load(
        modelPath,
        function (gltf) {
          const model = gltf.scene;
          scene.add(model);

          model.position.set(0, 0, 0);
          model.scale.set(1.25, 1.25, 1.25);

          if (gltf.animations && gltf.animations.length > 0) {
            mixer = new THREE.AnimationMixer(model);
            gltf.animations.forEach((animation) => {
              const action = mixer.clipAction(animation);
              action.play();
            });
          }
        },
        undefined,
        function (error) {
          console.error(error);
        }
    );

    camera.position.set(0, 1, 7);

    const clock = new THREE.Clock();

    function animate() {
      requestAnimationFrame(animate);

      if (mixer) {
        const delta = clock.getDelta();
        mixer.update(delta);
      }

      renderer.render(scene, camera);
    }

    animate();
  }

  // Load the model when the component is mounted (loaded)
  onMounted(() => {
    if (container.value) {
      loadFoxDesktopModel(container.value, "/animation-fox-with-bg.glb");
    }
  });
</script>

<style scoped>
  #model-fox-desktop-en {
    width: 100%;
    height: 100%;
    position: relative;
    overflow: hidden;
  }
</style>