<template>
  <div>
    <div>
      haha
    </div>
    <canvas id="canvas"></canvas>
  </div>
</template>

<script>
  import * as THREE from 'three';

  import axisGridHelper from "../../components/axisGridHelper";

  export default {

    data() {
      return {
        objects: [],
        scene: null,
        renderer: null,
        camera: null,
        gui:null
      }
    },
    mounted() {
      if(process.client){
        this.gui = require('dat.gui');
        this.gui = new this.gui.GUI();
      }
      this.init();
      this.render();
    },
    methods: {
      init() {
        const radius = 1;
        const widthSegment = 6;
        const heightSegment = 6;
        const canvas = document.querySelector('#canvas')
        this.renderer = new THREE.WebGLRenderer({canvas});

        // scene
        this.scene = new THREE.Scene();


        {
          // light
          const color = 0xFFFFF;
          const intensity = 3;
          const light = new THREE.PointLight(color, intensity);
          this.scene.add(light);
        }


        // camera
        this.camera = new THREE.PerspectiveCamera(40, window.innerWidth / window.innerHeight, 0.1, 1000);
        this.camera.position.set(0, 50, 0);
        this.camera.up.set(0, 0, 1);
        this.camera.lookAt(0, 0, 0);


        const sphereGeometry = new THREE.SphereBufferGeometry(radius, widthSegment, heightSegment);

        //create solar system
        const solarSystem = new THREE.Object3D();
        this.scene.add(solarSystem);
        this.objects.push(solarSystem);


        // add sun
        const sunMaterial = new THREE.MeshPhongMaterial({emissive: 0xFFFF00});
        const sunMesh = new THREE.Mesh(sphereGeometry, sunMaterial);
        sunMesh.scale.set(5, 5, 5);
        solarSystem.add(sunMesh);
        this.objects.push(sunMesh);

        // add earth orbit
        const earthOrbit = new THREE.Object3D();
        earthOrbit.position.x = 10;
        solarSystem.add(earthOrbit);
        this.objects.push(earthOrbit)

        // add earth
        const earthMaterial = new THREE.MeshPhongMaterial({color:223322 , emissive:0x112244})
        const earthMesh = new THREE.Mesh(sphereGeometry,earthMaterial);
        earthOrbit.add(earthMesh);
        this.objects.push(earthMesh);

        // add moon orbit
        const moonOrbit = new THREE.Object3D();
        moonOrbit.position.x = 2;
        this.objects.push(moonOrbit);
        earthOrbit.add(moonOrbit);

        //add moon
        const moonMaterial = new THREE.MeshPhongMaterial({color:0x111111,emissive:0x7777777})
        const moonMesh = new THREE.Mesh(sphereGeometry,moonMaterial);
        moonMesh.scale.set(.5,.5,.5);
        moonOrbit.add(moonMesh);
        this.objects.push(moonMesh);

        this.makeAxisGrid(solarSystem, 'solarSystem', 25);
        this.makeAxisGrid(sunMesh, 'sunMesh');
        this.makeAxisGrid(earthOrbit, 'earthOrbit');
        this.makeAxisGrid(earthMesh, 'earthMesh');
        this.makeAxisGrid(moonOrbit, 'moonOrbit');
        this.makeAxisGrid(moonMesh, 'moonMesh');
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
      makeAxisGrid(node,label,units){
        const helper = new axisGridHelper(node , units);
        this.gui.add(helper,'visible').name(label);
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
  canvas{
    width: 100%;
    height: 100%;
  }
</style>
