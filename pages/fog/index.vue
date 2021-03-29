<template>
  <div>
    <canvas id="canvas"></canvas>
  </div>
</template>

<script>
  import * as THREE from 'three';
  import FogGUIHelper from "../../components/fogGuiHelper";

  export default {

    data() {
      return {
        objects: [],
        scene: null,
        renderer: null,
        camera: null,
        gui: null
      }
    },
    mounted() {
      if (process.client) {
        this.gui = require('dat.gui');
        this.gui = new this.gui.GUI();
      }
      this.init();
      this.render();
    },
    methods: {
      init() {

        const canvas = document.querySelector('#canvas')
        this.renderer = new THREE.WebGLRenderer({canvas});
        this.scene = new THREE.Scene();


        // camera
        const fov = 75;
        const aspect = 2;
        const near = 1;
        const far = 10;
        this.camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
        this.camera.position.z = 2;

        {
          const near = 1;
          const far = 2;
          const color = 'lightblue';
          this.scene.fog = new THREE.Fog(color, near, far);
          this.scene.background = new THREE.Color(color);
        }
        const fogGUIHelper = new FogGUIHelper(this.scene.fog);
        this.gui.add(fogGUIHelper, 'near', near, far).listen();
        this.gui.add(fogGUIHelper, 'far', near, far).listen();

        const box = 1;
        const geometry = new THREE.BoxGeometry(box, box, box);


        this.objects = [
          this.makeBoxes(geometry, 0x44aa88, 0),
          this.makeBoxes(geometry, 0x8844aa, -2),
          this.makeBoxes(geometry, 0xaa8844, 2),
        ];

      },
      makeBoxes(geometry, color, x) {
        const material = new THREE.MeshPhongMaterial({color});
        const cube = new THREE.Mesh(geometry, material);
        this.scene.add(cube);
        cube.position.x = x;
        return cube
      },
      resizeRendererToDisplaySize(renderer) {
        const canvas = renderer.domElement;
        const width = canvas.clientWidth;
        const height = canvas.clientHeight;
        const needResize = canvas.width !== width || canvas.height !== height;
        if (needResize) {
          renderer.setSize(width, height, false);
        }
        return needResize;
      },
      render() {

        if (this.resizeRendererToDisplaySize(this.renderer)) {
          const canvas = this.renderer.domElement;
          this.camera.aspect = canvas.clientWidth / canvas.clientHeight;
          this.camera.updateProjectionMatrix();
        }
        this.objects.forEach((obj) => {
          obj.rotation.y += 0.01;
        });

        this.renderer.render(this.scene, this.camera);

        requestAnimationFrame(this.render);

      }
    }
  }
</script>

<style>
  canvas {
    width: 100%;
    height: 100%;
  }
</style>
