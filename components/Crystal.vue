<template>
  <div class='crystal' ref='container' :width='width' :height='height'></div>
</template>

<script>
import _ from 'lodash'
import * as THREE from 'three'
const vertexShader = require('../assets/crystal.vert')
const fragmentShader = require('../assets/crystal.frag')
import textureImage from '../assets/texture1.jpg'

export default {
  name: 'crystal',
  // each crystal requires color, size, faces
  props: ['data', 'crystalWidth', 'crystalHeight'],
  data() {
    return {
      width: this.data.length * this.crystalWidth,
      height: this.crystalHeight,
    }
  },
  created() {
    this.scene = new THREE.Scene()
    this.camera = new THREE.PerspectiveCamera(45, this.width / this.height, 1, 1000)
    this.renderer = new THREE.WebGLRenderer({antialias: true, alpha: true})

    // WebGL background color
    this.renderer.setClearColor(0xffffff, 0)

    // set renderer size
    this.renderer.setSize(this.width, this.height)

    // set camera position
    this.camera.position.set( 0, 0, 5 )
    this.camera.lookAt( 0, 0, 0 )

    // texture map, adapted from
    // https://gist.github.com/mattdesl/d74525cf21a9755383651289c799ac56
    this.textureMap = new THREE.TextureLoader().load(textureImage, texture => {
      this.renderer.render(this.scene, this.camera)
    })
  },
  mounted() {
    this.$refs.container.appendChild(this.renderer.domElement)
    this.renderData()

    this.renderer.render(this.scene, this.camera)
  },
  methods: {
    renderData() {
      const padding = 0.5
      const width = _.sumBy(this.data, d => d.size + padding)
      let x = 0
      _.each(this.data, (d, i) => {
        const {faces, size, color} = d

        x += size / 2
        const crystal = this.createCrystal(faces, color)
        crystal.scale.set(size * 0.5, size, size * 0.5)
        // crystal.scale.set(size, size, size)
        crystal.position.set(x + i * padding - width / 2, 0, 0)
        crystal.castShadow = true

        this.scene.add(crystal)

        x += size / 2
      })
    },
    createCrystal: function(numFaces, color) {
      const geometry = new THREE.SphereGeometry( 1, numFaces, 4 )
      // jitter vertices
      _.each(geometry.vertices, v => {
        v.x += _.random(-0.1, 0.1)
        v.z += _.random(-0.1, 0.1)
      })
      geometry.computeFlatVertexNormals()

      const material = new THREE.ShaderMaterial({
        flatShading: true,
        vertexShader,
        fragmentShader,
        side: THREE.DoubleSide,
        uniforms: {
          colorType: {value: color},
          textureMap: {value: this.textureMap},
        },
      })

      return new THREE.Mesh(geometry, material)
    },
  }
}
</script>

<style scoped>
</style>
