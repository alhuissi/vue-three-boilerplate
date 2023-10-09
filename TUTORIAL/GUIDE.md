# Guide


## 0. Prerequisites

Node:
https://nodejs.org/en/

Vue:
```
npm i -g @vue/cli
```

## 1. Project setup

Create new project:
```
vue create vue-3d-gallery
```

ThreeJS:
```
npm i three
```

Download 3D models

Original: Landscape gallery by @stoneysteiner:
https://sketchfab.com/3d-models/landscape-gallery-by-stoneysteiner-3702735762544e5796be4740cb6d5efc

![alt text](https://raw.githubusercontent.com/alhuissi/vue-three-boilerplate/master/TUTORIAL/sketchfab-screenshot.png)

Extract on /public

Compress /textures:
https://tinypng.com/

Project ready:
![alt text](https://github.com/alhuissi/3D-Virtual-Gallery/blob/main/project-ready-to-start.png)

## 2. Create scene


```
npm serve
```
Vue fresh new project:
![alt text](https://github.com/alhuissi/3D-Virtual-Gallery/blob/main/vue-new-project.png)

Delete default component and logo

Create new component VirtualGallery and add it to App.vue

![alt text](https://github.com/alhuissi/3D-Virtual-Gallery/blob/main/vue-new-project-2.png)

Add canvas on html template:
```
<template>
    <section>
        Virtual Gallery
        <canvas id="myCanvas"></canvas>
        <div id="blocker"></div>
    </section>
</template>
```

Import Three and set scene
```
<script>
import * as Three from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
//import { PointerLockControls } from "three/examples/jsm/controls/PointerLockControls.js";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";

export default {
    data() {
        return {
            camera: null,
            scene: null,
            renderer: null,
            controls: null,
            model: null,
            blocker: null,
            objects: [],
        }
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
            );
            this.scene = new Three.Scene()
            let light = new Three.HemisphereLight(0xffffff, 0x777788, 1)
            light.position.set(0, 0, 0)
            this.scene.add(light)

            let light3 = new Three.PointLight(0xffffff, 1, 250)
            light3.position.set(0, 10, 10)
            this.scene.add(light3)

            this.scene.background = new Three.Color(0x000000)

            const loader = new GLTFLoader()

            //Import 3D model
            loader.load(
                "/landscape_gallery_by_stoneysteiner/scene.gltf",
                (gltf) => {
                this.scene.add(gltf.scene);
                gltf.scene.scale.set(10, 10, 10) //scale here!
                gltf.scene.position.set(0, 0, 0) //position here!
                this.objects.push(gltf.scene)
                this.model = gltf.scene.children[0]
                },
                undefined,
                undefined
            )

            this.camera.position.y = 0
            this.camera.position.z = 10
            this.controls = new OrbitControls(this.camera, this.renderer.domElement)
            //this.controls = new PointerLockControls(this.camera, document.body);
            //this.scene.add(this.controls.getObject());

            this.renderer.setPixelRatio(window.devicePixelRatio)
            this.renderer.setSize(window.innerWidth, window.innerHeight)
            this.blocker.appendChild(this.renderer.domElement)
        },
        animate() {
            requestAnimationFrame(this.animate);
            //this.model.rotation.z += 0.001;
            this.renderer.render(this.scene, this.camera)
        },
        onWindowResize: function() {
            this.camera.aspect = window.innerWidth / window.innerHeight
            this.camera.updateProjectionMatrix()
            this.renderer.setSize(window.innerWidth, window.innerHeight)
            //this.composer.setSize(window.innerWidth, window.innerHeight)
        },
    },
    mounted() {
        this.init()
        this.animate()
        window.addEventListener("resize", this.onWindowResize, false)
    }
}
</script>
```

Style:
```
<style>
#blocker {
  position: fixed;
  top: 0%;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0);
  z-index: 2;
  transition: 925ms;
}

#myCanvas {
    position: absolute;
    top: 0%;
    width: 100%;
    height: 100%;
  }
</style>
```

![alt text](https://github.com/alhuissi/3D-Virtual-Gallery/blob/main/vue-virtual-gallery-1.png)

Add custom images

	Change the 12 images. 1024x1024

## 3. Add physics and movement

	Make the room rotate at start/pause

 ## 4. Play!

