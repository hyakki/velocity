<template>
  <div class="webgl-component" ref="container"></div>
</template>

<script lang="ts">
/* eslint-disable @typescript-eslint/no-unused-vars */

import { onMounted, ref, onUnmounted } from 'vue'
import * as THREE from 'three'

import glsl from 'glslify'

import city from './textures/city.png'

import { lerp } from 'math-toolbox'
import Hammer from 'hammerjs'
import mitt from '@/mitt/mitt'

// eslint-disable-next-line
let OrbitControls = require('three-orbit-controls')(THREE)

export default {
  name: 'Webgl',
  setup() {
    const container = ref()

    let time = 0
    let camera, light, scene, renderer
    let sketchGeometry, sketchMaterial, sketchMesh
    let dragGeometry, dragMaterial, dragMesh
    let velocityHelper, accHelper, dragHelper
    let hammer

    const velocity = new THREE.Vector3(0.0, 0.0, 0.0)
    const acc = new THREE.Vector3(0.0, 0.0, 0.0)
    const drag = new THREE.Vector3(0.0, 0.0, 0.0)

    const setSize = () => {
      const { width, height } = container.value.getBoundingClientRect()

      renderer.setSize(width, height)
    }

    const mouseWheelHandler = e => {
      const { deltaY } = e

      acc.x += deltaY > 0 ? -0.02 : 0.02
    }

    const setCameraAspect = () => {
      const { width, height } = container.value.getBoundingClientRect()

      camera.aspect = width / height
      camera.updateProjectionMatrix()
    }

    const createSketch = (addToScene = true) => {
      const pivot = new THREE.Object3D()

      sketchGeometry = new THREE.BoxBufferGeometry(1, 1, 1, 1, 1)
      sketchGeometry.translate(0, 0.5, 0)

      sketchMaterial = new THREE.MeshPhongMaterial({
        color: 0xdddddd,
        side: THREE.DoubleSide,
      })

      sketchMesh = new THREE.Mesh(sketchGeometry, sketchMaterial)

      pivot.add(sketchMesh)

      addToScene && scene.add(pivot)
    }

    const createCamera = () => {
      const { width, height } = container.value.getBoundingClientRect()

      camera = new THREE.PerspectiveCamera(70, width / height, 0.001, 100)
      camera.position.x = 1.8
      camera.position.y = 2.7
      camera.position.z = 2.8
      camera.lookAt(0, 0, 0)
    }

    const createLight = () => {
      light = new THREE.SpotLight(0xffffff)
      light.position.set(2, 3, 1)
      light.castShadow = true
      light.intensity = 1.5

      scene.add(light)
    }

    const createScene = () => {
      scene = new THREE.Scene()
      scene.background = new THREE.Color(0x111111)
    }

    const createRenderer = () => {
      const { width, height } = container.value.getBoundingClientRect()

      renderer = new THREE.WebGLRenderer({ antialias: true })
    }

    const createOrbitControls = () => {
      new OrbitControls(camera, renderer.domElement)
    }

    const createVelocityHelper = () => {
      const dir = velocity
      const origin = new THREE.Vector3(0, 0, 1.25)
      const length = 0.5
      const hex = 0xff0000

      velocityHelper = new THREE.ArrowHelper(dir, origin, length, hex)

      scene.add(velocityHelper)
    }

    const createAccHelper = () => {
      const dir = acc
      const origin = new THREE.Vector3(0, 0, 1.5)
      const length = 0.5
      const hex = 0x00ff00

      accHelper = new THREE.ArrowHelper(dir, origin, length, hex)

      scene.add(accHelper)
    }

    const createDragHelper = () => {
      const dir = drag
      const origin = new THREE.Vector3(0, 0, 1.75)
      const length = 0.5
      const hex = 0x0000ff

      dragHelper = new THREE.ArrowHelper(dir, origin, length, hex)

      scene.add(dragHelper)
    }

    const createGridHelper = () => {
      const gridHelper = new THREE.GridHelper(30, 2)

      scene.add(gridHelper)
    }

    const deccelerationCoef = 0.5
    const dragCoef = 0.1

    const update = () => {
      time += 1

      acc.x = lerp(0.0, acc.x, 1 - deccelerationCoef)
      drag.x = velocity.x * -1 * dragCoef

      velocity.add(acc)
      velocity.add(drag)

      // velocity.x = lerp(0.0, velocity.x, 0.2)

      velocityHelper.setDirection(velocity.clone().normalize())
      velocityHelper.setLength(velocity.clone().length() * 30.0)

      accHelper.setDirection(acc.clone().normalize())
      accHelper.setLength(acc.clone().length() * 30.0)

      dragHelper.setDirection(drag.clone().normalize())
      dragHelper.setLength(drag.clone().length() * 30.0)

      sketchMesh.scale.y += velocity.x

      renderer.render(scene, camera)

      mitt.emit('velocity', Math.round(velocity.length() * 100.0 * 100) / 100)
      mitt.emit('acceleration', Math.round(acc.length() * 100.0 * 100) / 100)
      mitt.emit('drag', Math.round(drag.length() * 100.0 * 100) / 100)

      window.requestAnimationFrame(update)
    }

    const viewportHandler = () => {
      setSize()
      setCameraAspect()
    }

    onMounted(() => {
      createCamera()
      createScene()
      createLight()

      createSketch()

      createRenderer()
      // createOrbitControls()
      createVelocityHelper()
      createAccHelper()
      createDragHelper()
      createGridHelper()

      setSize()
      container.value.appendChild(renderer.domElement)

      update()

      hammer = new Hammer.Manager(container.value)
      hammer.add(
        new Hammer.Pan({ direction: Hammer.DIRECTION_VERTICAL, threshold: 0 })
      )

      hammer.on('pan', e => {
        if (
          e.overallVelocityY === Infinity ||
          e.overallVelocityY === -Infinity
        ) {
          console.log('Who cares about infinity?')
          return
        }

        acc.x = (e.overallVelocityY / 100.0) * -1
      })

      window.addEventListener('mousewheel', mouseWheelHandler)
      window.addEventListener('resize', viewportHandler)
    })

    onUnmounted(() => {
      hammer.destroy()

      window.removeEventListener('resize', viewportHandler)
      window.removeEventListener('mousewheel', mouseWheelHandler)
    })

    return {
      container,
    }
  },
}
</script>
