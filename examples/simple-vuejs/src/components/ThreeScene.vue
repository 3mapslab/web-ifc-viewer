<template>
  <div class="canvas-container">
    <v-layout
      align-center
      justify-center
      column
      fill-height
      style="position: absolute; width: 100%; background-color: rgba(255, 255, 255, 0.95)"
      v-if="loading"
    >
      <v-flex row align-center>
        <v-progress-circular indeterminate :size="150" :width="6" color="grey">
          <div class="overline" style="font-size: 0.65rem !important">
            <v-icon left>mdi-cube-outline</v-icon>
            <span>loading...</span>
          </div>
        </v-progress-circular>
      </v-flex>
    </v-layout>
    <canvas class="canvas" ref="canvas"></canvas>
  </div>
</template>
<script>
import {
  AmbientLight,
  Color,
  DirectionalLight,
  GridHelper,
  PerspectiveCamera,
  Scene,
  WebGLRenderer
} from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import { IFCLoader } from 'three/examples/jsm/loaders/IFCLoader';

export default {
  props: ['resize', 'ifc'],
  data() {
    return {
      loading: false,
      camera: null,
      renderer: null,
      scene: null,
      controls: null,
      ifcLoader: null,
      loadedIFCGeometry: null
    };
  },
  watch: {
    resize() {
      this.handleComponentResized();
    },
    ifc() {
      this.loadIfcFile();
    }
  },
  mounted() {
    this.$nextTick(() => {
      this.initScene();
      //Adjust the viewport to the size of the browser
      window.addEventListener('resize', () => {
        this.handleComponentResized();
      });
      this.loadIfcFile();
    });
  },
  beforeDestroy() {},
  methods: {
    initScene() {
      //Creates the Three.js scene
      this.scene = new Scene();
      this.scene.background = new Color(0xaaaaaa);

      //Creates the camera (point of view of the user)
      this.camera = new PerspectiveCamera(
        75,
        this.$refs.canvas.clientWidth / this.$refs.canvas.clientHeight
      );
      this.camera.position.z = 15;
      this.camera.position.y = 13;
      this.camera.position.x = 8;

      //Creates the lights of the scene
      const lightColor = 0xffffff;

      const ambientLight = new AmbientLight(lightColor, 0.5);
      this.scene.add(ambientLight);

      const directionalLight = new DirectionalLight(lightColor, 1);
      directionalLight.position.set(0, 10, 0);
      directionalLight.target.position.set(-5, 0, 0);
      this.scene.add(directionalLight);
      this.scene.add(directionalLight.target);

      //Sets up the renderer, fetching the canvas of the HTML
      this.renderer = new WebGLRenderer({ canvas: this.$refs.canvas });
      this.renderer.setSize(this.$refs.canvas.clientWidth, this.$refs.canvas.clientHeight);
      this.renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

      //Creates grids and axes in the scene
      this.grid = new GridHelper(200, 50);

      //Creates the orbit controls (to navigate the scene)
      this.controls = new OrbitControls(this.camera, this.$refs.canvas);
      this.controls.enableDamping = false;
      this.controls.target.set(-2, 0, 0);

      //Sets up the IFC loading
      this.ifcLoader = new IFCLoader();
      this.ifcLoader.setWasmPath('wasm/');

      //Animation loop
      this.animate();
    },
    animate() {
      this.controls.update();
      this.renderer.render(this.scene, this.camera);
      requestAnimationFrame(this.animate);
    },

    //#endregion
    handleComponentResized() {
      this.$nextTick(() => {
        if (this.$refs.canvas) {
          this.camera.aspect = this.$refs.canvas.clientWidth / this.$refs.canvas.clientHeight;
          this.camera.updateProjectionMatrix();
          this.renderer.setSize(this.$refs.canvas.clientWidth, this.$refs.canvas.clientHeight);
        }
      });
    },

    clearScene() {
      if (this.scene && this.loadedIFCGeometry) {
        this.scene.remove(this.loadedIFCGeometry);
      }
    },

    loadIfcFile() {
      this.loading = true;
      this.clearScene();
      if (this.ifc) {
        this.ifcLoader.load(window.URL.createObjectURL(this.ifc), (geometry) => {
          this.loadedIFCGeometry = geometry;
          this.scene.add(this.loadedIFCGeometry);
          this.loading = false;
        });
      } else {
        this.loading = false;
      }
    }
  }
};
</script>
<style scoped>
.canvas-container,
.canvas {
  width: 100%;
  height: 100%;
}
</style>