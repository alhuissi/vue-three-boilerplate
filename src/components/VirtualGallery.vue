<template>
  <section>
    <canvas id="myCanvas"></canvas>
    <div id="blocker"></div>
    <div>
      <button id="boton-play" @click="onClickInstructions()">PLAY</button>
    </div>
  </section>
</template>

<script>
import * as Three from "three"
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js"
import { PointerLockControls } from "three/examples/jsm/controls/PointerLockControls.js"
//import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js"

export default {
  data() {
    return {
      camera: null,
      scene: null,
      renderer: null,
      controls: null,
      model: null,
      blocker: null,
      raycaster: null,
      moveForward: false,
      moveBackward: false,
      moveLeft: false,
      moveRight: false,
      running: false,
      canJump: false,
      prevTime: performance.now(),
      velocity: null,
      direction: null,
      objects: [],
      memes: [],
    };
  },
  methods: {
    init() {
      this.blocker = document.getElementById("blocker")
      const canvas = document.querySelector("#myCanvas")
      this.renderer = new Three.WebGLRenderer({ canvas, antialias: true })
      this.camera = new Three.PerspectiveCamera(
        75,
        window.innerWidth / window.innerHeight,
        0.1,
        1000
      )
      this.scene = new Three.Scene()
      let light = new Three.HemisphereLight(0xffffff, 0x777788, 1)
      light.position.set(0, 0, 0)
      this.scene.add(light)

      let light3 = new Three.PointLight(0xffffff, 1, 250)
      light3.position.set(0, 10, 10)
      this.scene.add(light3)

      this.scene.background = new Three.Color(0x000000)

      this.velocity = new Three.Vector3()
      this.direction = new Three.Vector3()

      this.raycaster = new Three.Raycaster(
        new Three.Vector3(),
        new Three.Vector3(0, -1, 0),
        0,
        10
      );

      const loader = new GLTFLoader()

      //Import 3D model
      loader.load(
        "/landscape_gallery_by_stoneysteiner/scene.gltf",
        (gltf) => {
          this.scene.add(gltf.scene)
          gltf.scene.scale.set(10, 10, 10)
          gltf.scene.position.set(0, 0, 0)
          this.objects.push(gltf.scene)
          this.model = gltf.scene.children[0]
        },
        undefined,
        undefined
      )

      this.camera.position.y = 0
      this.camera.position.z = 10
      //this.controls = new OrbitControls(this.camera, this.renderer.domElement)
      this.controls = new PointerLockControls(this.camera, document.body)
      this.scene.add(this.controls.getObject())

      this.renderer.setPixelRatio(window.devicePixelRatio)
      this.renderer.setSize(window.innerWidth, window.innerHeight)
      this.blocker.appendChild(this.renderer.domElement)
    },
    onLock: function () {
      this.blocker.style.display = ""
      document.getElementById("boton-play").style.opacity = "0"
    },
    onUnlock: function () {
      if (this.goku) {
        this.hablar()
      } else {
        this.blocker.style.display = "block"
        document.getElementById("boton-play").style.opacity = "1"
      }
    },
    onClickInstructions: function () {
      this.controls.lock()
    },
    animate() {
      let time = performance.now()
      requestAnimationFrame(this.animate)
      //this.model.rotation.z += 0.001;
      if (this.controls.isLocked === true) {
        this.raycaster.ray.origin.copy(this.controls.getObject().position)
        this.raycaster.ray.origin.y -= 14

        let intersections = this.raycaster.intersectObjects(this.objects)

        let onObject = intersections.length > 0

        let delta = (time - this.prevTime) / 1000

        this.velocity.x -= this.velocity.x * 10.0 * delta
        this.velocity.z -= this.velocity.z * 10.0 * delta

        this.velocity.y -= 9.8 * 100.0 * delta // 100.0 = mass

        this.direction.z = Number(this.moveForward) - Number(this.moveBackward)
        this.direction.x = Number(this.moveRight) - Number(this.moveLeft)
        this.direction.normalize() // this ensures consistent movements in all directions

        //Controles y velocidad
        if (this.moveForward || this.moveBackward) {
          if (this.running) {
            this.velocity.z -= this.direction.z * 1400.0 * delta
          } else {
            this.velocity.z -= this.direction.z * 700.0 * delta
          }
        }
        if (this.moveLeft || this.moveRight) {
          if (this.running) {
            this.velocity.x -= this.direction.x * 1400.0 * delta
          } else {
            this.velocity.x -= this.direction.x * 700.0 * delta
          }
        }

        //Colisiones
        if (onObject === true) {
          this.velocity.y = Math.max(0, this.velocity.y)
          this.canJump = true
        }
        if (this.controls.getObject().position.z > 35) {
            this.velocity.z = 0
            this.controls.getObject().position.z = 35
        }
        if (this.controls.getObject().position.z < -35) {
            this.velocity.z = 0
            this.controls.getObject().position.z = -35
        }
        if (this.controls.getObject().position.x > 35) {
            this.velocity.x = 0
            this.controls.getObject().position.x = 35
        }
        if (this.controls.getObject().position.x < -35) {
            this.velocity.x = 0
            this.controls.getObject().position.x = -35
        }

        this.controls.moveRight(-this.velocity.x * delta)
        this.controls.moveForward(-this.velocity.z * delta)

        this.controls.getObject().position.y += this.velocity.y * delta // new behavior

        if (this.controls.getObject().position.y < 0) {
          this.velocity.y = 0
          this.controls.getObject().position.y = 0
          this.canJump = true
        }

        this.prevTime = time
      } else {
        //this.camera.position.z += 0.1
      }
      this.renderer.render(this.scene, this.camera)
    },
    onWindowResize: function () {
      this.camera.aspect = window.innerWidth / window.innerHeight
      this.camera.updateProjectionMatrix()
      this.renderer.setSize(window.innerWidth, window.innerHeight)
      //this.composer.setSize(window.innerWidth, window.innerHeight)
    },
    fetchMemes() {
      fetch("https://meme-api.herokuapp.com/gimme/12")
        .then((response) => response.json())
        .then((data) => console.log(data))
    },
    onKeyDown: function (event) {
      switch (event.keyCode) {
        case 38: // up
        case 87: // w
          this.moveForward = true
          break

        case 37: // left
        case 65: // a
          this.moveLeft = true
          break

        case 40: // down
        case 83: // s
          this.moveBackward = true
          break

        case 39: // right
        case 68: // d
          this.moveRight = true
          break

        case 16: // shift
          this.running = true
          break

        case 32: // space
          if (this.canJump === true) this.velocity.y += 150
          this.canJump = false
          break

      }
    },
    onKeyUp: function (event) {
      switch (event.keyCode) {
        case 38: // up
        case 87: // w
          this.moveForward = false;
          break;

        case 37: // left
        case 65: // a
          this.moveLeft = false;
          break;

        case 40: // down
        case 83: // s
          this.moveBackward = false;
          break;

        case 39: // right
        case 68: // d
          this.moveRight = false;
          break;

        case 16: // shift
          this.running = false;
          break;
      }
    },
  },
  mounted() {
    this.init();
    this.animate();
    window.addEventListener("resize", this.onWindowResize, false);
    document.addEventListener("keydown", this.onKeyDown, false);
    document.addEventListener("keyup", this.onKeyUp, false);
    this.controls.addEventListener("lock", this.onLock, false);
    this.controls.addEventListener("unlock", this.onUnlock, false);
    this.fetchMemes();
  },
  destroyed() {
    this.renderer.forceContextLoss();
    window.removeEventListener("resize", this.onWindowResize, false);
    document.removeEventListener("keydown", this.onKeyDown, false);
    document.removeEventListener("keyup", this.onKeyUp, false);
    this.controls.removeEventListener("lock", this.onLock, false);
    this.controls.removeEventListener("unlock", this.onUnlock, false);
  },
};
</script>

<style>
#blocker {
  position: fixed;
  top: 0%;
  left: 0%;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0);
  z-index: 2;
  transition: 925ms;
}

#myCanvas {
  position: absolute;
  top: 0%;
  left: 0%;
  width: 100%;
  height: 100%;
}

#boton-play {
  position: fixed;
  text-align: center;
  cursor: pointer;
  font-size: calc(16px + 0.6vw);
  z-index: 999;
  transition: 325ms ease;
  opacity: 1;
}

</style>