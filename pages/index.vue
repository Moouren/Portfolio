<template>
  <canvas id="canvas"></canvas>
</template>

<script>
  import * as THREE from 'three'

  export default {
    data() {
      return {
        scene: null,
        camera: null,
        renderer: null,
        objs: [],
      }
    },
    mounted() {
      this.init();
      this.render();
    },
    methods: {


      makeInstance(color, geometry, x) {
        const material = new THREE.MeshPhongMaterial({color: color})
        const cube = new THREE.Mesh(geometry, material)
        cube.position.x = x * 3;
        this.scene.add(cube);

        return cube;
      },

      addObject(x, y, obj) {
        obj.position.x = x * 15;
        obj.position.y = y * 15;
        this.scene.add(obj);
        this.objs.push(obj)
      },
      rotateCubes(ndx) {

        this.cubes[ndx].rotation.x += 0.1;
        this.cubes[ndx].rotation.y += 0.1 * ndx;
      },
      resizeRendererToDisplaySize() {
        const canvas = this.renderer.domElement;
        const pixelRatio = window.devicePixelRatio;
        const width = canvas.clientWidth * pixelRatio | 0;
        const height = canvas.clientHeight * pixelRatio | 0;
        const needResize = canvas.width !== width || canvas.height !== height;
        if (needResize) {
          this.renderer.setSize(width, height, false);
        }
        return needResize;
      },
      createMaterial() {
        const material = new THREE.MeshPhongMaterial({
          side: THREE.DoubleSide
        });
        const hue = Math.random();
        const saturation = 1;
        const luminance = .5;
        material.color.setHSL(hue, saturation, luminance);
        return material;

      },
      addSolidGeometry(x, y, geometry) {
        const mesh = new THREE.Mesh(geometry, this.createMaterial());
        this.addObject(x, y, mesh);
      },
      init() {
        const canvas = document.querySelector('#canvas')
        this.scene = new THREE.Scene();
        this.camera = new THREE.PerspectiveCamera(40, window.innerWidth / window.innerHeight, 1, 1000);
        this.camera.position.set(0, 0, 100);
        this.camera.lookAt(0, 0, 0);
        this.renderer = new THREE.WebGLRenderer({canvas});
        this.renderer.setSize(window.innerWidth, window.innerHeight);
        const geometry = new THREE.BoxGeometry(10, 10, 10);
        {
          const width = 8;
          const height = 8;
          const depth = 8;
          this.addSolidGeometry(-2, -2, new THREE.BoxBufferGeometry(width, height, depth));
        }


        // Light
        const color = 0xFFFFFF;
        const intensity = 1;
        const light = new THREE.DirectionalLight(color, intensity);
        light.position.set(-1, 2, 4);
        this.scene.add(light);
      },
      render() {
        requestAnimationFrame(this.render);

        if (this.resizeRendererToDisplaySize) {
          const canvas = this.renderer.domElement;
          this.camera.aspect = canvas.clientWidth / canvas.clientHeight;
          this.camera.updateProjectionMatrix();
        }


        this.renderer.render(this.scene, this.camera);
      }
    }
    ,
  }
</script>
<style>
  body {
    margin: 0;
    padding: 0;
  }

  canvas {
    width: 100% !important;
    height: 100vh !important;
    background-color: #000;
  }
</style>

