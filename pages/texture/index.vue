<template>
  <div>
    <canvas id="canvas">
    </canvas>
    <div class="split">
      <div id="view1" tabindex="1"></div>
      <div id="view2" tabindex="2"></div>
    </div>
    <div id="loading" v-if="loading !== 1">
      <div class="progress">
        <div class="progressbar"></div>
      </div>
    </div>
  </div>

</template>

<script>
  import * as THREE from 'three'

  const OrbitControls = require('three-orbit-controls')(THREE);
  import minMaxGuiHelper from '../../components/minMaxGuiHelper';


  export default {

    data() {
      return {
        scene: null,
        object: [],
        renderer: null,
        camera: null,
        canvas: null,
        cameraHelper: null,
        camera2: null,
        view1Elem: null,
        view2Elem: null,
        loading: 0,
        gui: null,
        RectAreaLightHelper: null,
        rectAreaUniformLib: null,
      }
    },
    mounted() {
      if (process.client) {
        this.gui = require('dat.gui');
        this.gui = new this.gui.GUI();
        this.RectAreaLightHelper = require('three/examples/jsm/helpers/RectAreaLightHelper.js');
        this.rectAreaUniformLib = require('three/examples/jsm/lights/RectAreaLightUniformsLib.js');
        this.init();
      }

      this.render();
    },
    methods: {

      //on resize change resolution and display size
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
      makeXYZGUI(gui, vector3, name, onChangeFn) {
        const folder = gui.addFolder(name);
        folder.add(vector3, 'x', -10, 10).onChange(onChangeFn);
        folder.add(vector3, 'y', 0, 10).onChange(onChangeFn);
        folder.add(vector3, 'z', -10, 10).onChange(onChangeFn);
        folder.open();
      },
      updateCamera() {
        this.camera.updateProjectionMatrix();
      },
      init() {
        this.canvas = document.querySelector('#canvas');
        this.scene = new THREE.Scene();
        this.view1Elem = document.querySelector('#view1');
        this.view2Elem = document.querySelector('#view2');


        this.cameraHelper = new THREE.CameraHelper(this.camera);
        // this.scene.add(this.cameraHelper);
        this.camera = new THREE.PerspectiveCamera(40, window.innerWidth / window.innerHeight, 0.1, 100);
        this.camera.position.set(0, 10, 20);

        const control = new OrbitControls(this.camera, this.view1Elem);
        control.target.set(0, 5, 0);
        control.update();

        // camera 2
        this.camera2 = new THREE.PerspectiveCamera(
          60,  // fov
          2,   // aspect
          0.1, // near
          500, // far
        );
        this.camera2.position.set(40, 10, 30);
        this.camera2.lookAt(0, 5, 0);

        const controls2 = new OrbitControls(this.camera2, this.view2Elem);
        controls2.target.set(0, 5, 0);
        controls2.update();

        this.renderer = new THREE.WebGLRenderer(this.canvas);
        // fix how light falls off as distance
        this.renderer.physicallyCorrectLights = true;
        this.renderer.setSize(window.innerWidth, window.innerHeight);
        const geometry = new THREE.BoxGeometry(1, 1, 1);

        const loaderManager = new THREE.LoadingManager();
        const loader = new THREE.TextureLoader(loaderManager);

        const materials = [
          new THREE.MeshBasicMaterial({map: loader.load('https://threejsfundamentals.org/threejs/resources/images/flower-1.jpg')}),
          new THREE.MeshBasicMaterial({map: loader.load('https://threejsfundamentals.org/threejs/resources/images/flower-2.jpg')}),
          new THREE.MeshBasicMaterial({map: loader.load('https://threejsfundamentals.org/threejs/resources/images/flower-3.jpg')}),
          new THREE.MeshBasicMaterial({map: loader.load('https://threejsfundamentals.org/threejs/resources/images/flower-4.jpg')}),
          new THREE.MeshBasicMaterial({map: loader.load('https://threejsfundamentals.org/threejs/resources/images/flower-5.jpg')}),
          new THREE.MeshBasicMaterial({map: loader.load('https://threejsfundamentals.org/threejs/resources/images/flower-6.jpg')}),
        ];

        const progressBarElem = document.querySelector('.progressbar');
        loaderManager.onLoad = () => {
          const cube = new THREE.Mesh(geometry, materials);
          cube.position.set(0, 1, 0);
          this.scene.add(cube);
          this.object.push(cube);
        }
        loaderManager.onProgress = (urlOfLastItemLoaded, itemsLoaded, itemsTotal) => {
          const progress = itemsLoaded / itemsTotal;
          progressBarElem.style.transform = `scaleX(${progress})`;
          this.loading = progress;
        };

        // light

        const color = 'white';
        const intensity = 1;

        const light = new THREE.RectAreaLight(color, intensity, 30, 16);
        light.power = 800;
        light.decay = 2;
        light.distance = Infinity;

        light.position.set(0, 10, 0);
        light.rotation.x = THREE.MathUtils.degToRad(-90)
        this.scene.add(light);
        const helper = new this.RectAreaLightHelper.RectAreaLightHelper(light);
        // this.scene.add(helper);
        this.rectAreaUniformLib.RectAreaLightUniformsLib.init();

        function updateLight() {
          // light.target.updateMatrixWorld();
          helper.update();
        }

        updateLight();


        // plane
        const planeSize = 40;
        const planeLoader = new THREE.TextureLoader();
        const texture = planeLoader.load("https://threejsfundamentals.org/threejs/resources/images/checker.png");
        texture.wrapS = THREE.RepeatWrapping;
        texture.wrapT = THREE.RepeatWrapping;
        texture.magFilter = THREE.NearestFilter;
        const repeat = planeSize / 2;
        texture.repeat.set(repeat, repeat);

        // plane mesh
        const planeGeo = new THREE.PlaneBufferGeometry(planeSize, planeSize)
        const planeMaterial = new THREE.MeshStandardMaterial({
          map: texture,
          side: THREE.DoubleSide
        })
        const mesh = new THREE.Mesh(planeGeo, planeMaterial);
        mesh.rotation.x = Math.PI * -.5;
        this.scene.add(mesh);


        // cube
        {
          const cubeSize = 4;
          const cubeGeo = new THREE.BoxBufferGeometry(cubeSize, cubeSize, cubeSize);
          const cubeMat = new THREE.MeshStandardMaterial({color: "#8AC"});
          const mesh = new THREE.Mesh(cubeGeo, cubeMat);
          mesh.position.set(cubeSize + 1, cubeSize / 2, 0);
          this.scene.add(mesh);
        }
        // sphere
        {
          const sphereRadius = 3;
          const sphereWidthDivision = 32;
          const sphereHeightDivision = 16;
          const sphereGeo = new THREE.SphereBufferGeometry(sphereRadius, sphereWidthDivision, sphereHeightDivision);
          const sphereMat = new THREE.MeshStandardMaterial({color: "#CA8"});
          const mesh = new THREE.Mesh(sphereGeo, sphereMat);
          mesh.position.set(-sphereRadius - 1, sphereRadius + 2, 0);
          this.scene.add(mesh);

        }
        // this.gui.addColor(new colorGuiHelper(light, 'color'), 'value').name('color');
        // this.gui.add(light, 'intensity', 0, 10, 0.01);
        // this.gui.add(light, 'width', 0, 20).onChange(updateLight);
        // this.gui.add(light, 'height', 0, 20).onChange(updateLight);
        // this.gui.add(new DegRadHelper(light.rotation, 'x'), 'value', -180, 180).name('x rotation').onChange(updateLight);
        // this.gui.add(new DegRadHelper(light.rotation, 'y'), 'value', -180, 180).name('y rotation').onChange(updateLight);
        // this.gui.add(new DegRadHelper(light.rotation, 'z'), 'value', -180, 180).name('z rotation').onChange(updateLight);
        //
        // this.gui.add(light, 'decay', 0, 4, 0.01);
        // this.gui.add(light, 'power', 0, 2000);
        //
        // this.makeXYZGUI(this.gui, light.position, 'position', updateLight);

        // this.gui.addColor(new colorGuiHelper(light, 'color'), 'value').name('color');
        // this.gui.add(light, 'decay', 0, 4, 0.01);
        // this.gui.add(light, 'power', 0, 1220);
        // this.makeXYZGUI(this.gui, light.position, 'position');


        // this.gui.add(this.camera, 'fov', 1, 180).onChange(this.updateCamera);
        // const minMaxGUIHelper = new minMaxGuiHelper(this.camera, 'near', 'far', 0.1);
        // this.gui.add(minMaxGUIHelper, 'min', 0.1, 50, 0.1).name('near').onChange(this.updateCamera);
        // this.gui.add(minMaxGUIHelper, 'max', 0.1, 50, 0.1).name('far').onChange(this.updateCamera);


        this.gui.add(this.camera, 'fov', 1, 180);
        const minMaxGUIHelper = new minMaxGuiHelper(this.camera, 'near', 'far', 0.1);
        this.gui.add(minMaxGuiHelper, 'min', 0.1, 50, 0.1).name('near');
        this.gui.add(minMaxGuiHelper, 'max', 0.1, 50, 0.1).name('far');

      },
      setScissorForElement(elem) {
        const canvasRect = this.canvas.getBoundingClientRect();
        const elemRect = elem.getBoundingClientRect();

        // compute a canvas relative rectangle
        const right = Math.min(elemRect.right, canvasRect.right) - canvasRect.left;
        const left = Math.max(0, elemRect.left - canvasRect.left);
        const bottom = Math.min(elemRect.bottom, canvasRect.bottom) - canvasRect.top;
        const top = Math.max(0, elemRect.top - canvasRect.top);

        const width = Math.min(canvasRect.width, right - left);
        const height = Math.min(canvasRect.height, bottom - top);

        // setup the scissor to only render to that part of the canvas
        const positiveYUpBottom = canvasRect.height - bottom;
        this.renderer.setScissor(left, positiveYUpBottom, width, height);
        this.renderer.setViewport(left, positiveYUpBottom, width, height);

        // return the aspect
        return width / height;
      },

      render() {
        requestAnimationFrame(this.render);

        // if (this.resizeRendererToDisplaySize) {
        //   const canvas = this.renderer.domElement;
        //   this.camera.aspect = canvas.clientWidth / canvas.clientHeight;
        //   this.camera.updateProjectionMatrix();
        // }

        this.resizeRendererToDisplaySize(this.renderer);

        // turn on the scissor
        this.renderer.setScissorTest(true);

        // render the original view
        {
          const aspect = this.setScissorForElement(this.view1Elem);

          // adjust the camera for this aspect
          this.camera.aspect = aspect;
          this.camera.updateProjectionMatrix();
          this.cameraHelper.update();

          // don't draw the camera helper in the original view
          this.cameraHelper.visible = false;

          this.scene.background.set(0x000000);

          // render
          this.renderer.render(this.scene, this.camera);
        }

        // render from the 2nd camera
        {
          const aspect = this.setScissorForElement(this.view2Elem);

          // adjust the camera for this aspect
          this.camera2.aspect = aspect;
          this.camera2.updateProjectionMatrix();

          // draw the camera helper in the 2nd view
          this.cameraHelper.visible = true;

          this.scene.background.set(0x000040);

          this.renderer.render(this.scene, this.camera2);
        }
        this.object.forEach((cube) => {
          cube.rotation.x += 0.01;
          cube.rotation.y += 0.01
        })

        this.renderer.render(this.scene, this.camera);
      }
    }
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

  #loading {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  #loading .progress {
    margin: 1.5em;
    border: 1px solid white;
    width: 50vw;
  }

  #loading .progressbar {
    margin: 2px;
    background: white;
    height: 1em;
    transform-origin: top left;
    transform: scaleX(0);
  }

  .split {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    display: flex;
  }

  .split > div {
    width: 100%;
    height: 100%;
  }
</style>

