<template>
  <div ref="modelContainer" class="model-container"></div>
</template>

<script>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'

export default {
  name: 'Model3D',
  mounted() {
    this.initThreeJS()
  },
  methods: {
    initThreeJS() {
      const container = this.$refs.modelContainer
      const width = container.clientWidth
      const height = container.clientHeight

      // Scène, caméra et renderer
      const scene = new THREE.Scene()
      const camera = new THREE.PerspectiveCamera(75, width / height, 0.1, 100000)
      camera.position.set(0, 2, 50)
      const renderer = new THREE.WebGLRenderer({ antialias: true })
      renderer.setSize(width, height)
      container.appendChild(renderer.domElement)

      // Lumière ambiante
      const ambientLight = new THREE.AmbientLight(0xffffff, 0.3) // Lumière ambiante
      scene.add(ambientLight)

      // Lumière directionnelle
      const directionalLight = new THREE.DirectionalLight(0xffffff, 1)
      directionalLight.position.set(5, 10, 5).normalize()
      scene.add(directionalLight)

      // OrbitControls
      const controls = new OrbitControls(camera, renderer.domElement)
      controls.enableDamping = true

      // Raycaster et souris
      const raycaster = new THREE.Raycaster()
      const mouse = new THREE.Vector2()

      // Charger le modèle
      const loader = new GLTFLoader()
      let dinosaur = null // Pour stocker le modèle du dinosaure

      loader.load(
        '/models/planet.gltf', // Ton modèle avec la planète et le dinosaure
        (gltf) => {
          const model = gltf.scene
          model.position.set(0, 0, 0)
          scene.add(model)

          // Assurer que les matériaux sont réactifs à la lumière
          model.traverse((child) => {
            if (child.isMesh) {
              child.material = new THREE.MeshStandardMaterial({
                color: child.material.color,
                roughness: 0.5,
                metalness: 0.5,
              })
            }
          })

          // Trouver le dinosaure dans le modèle Sphère
          dinosaur = model.getObjectByName('0') // Remplace 'Sphère' par le nom exact dans le modèle
          if (dinosaur) {
            dinosaur.userData = { clickable: true } // Marquer le dinosaure comme cliquable
          }
        },
        undefined,
        (error) => {
          console.error('Erreur lors du chargement du modèle :', error)
        },
      )

      // Écouter les clics
      container.addEventListener('click', (event) => {
        const rect = container.getBoundingClientRect()
        mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1
        mouse.y = -((event.clientY - rect.top) / rect.height) * 2 + 1
        raycaster.setFromCamera(mouse, camera)

        const intersects = raycaster.intersectObjects(scene.children, true)
        if (intersects.length > 0) {
          const intersected = intersects[0].object
          if (intersected.userData.clickable) {
            console.log('Dinosaure cliqué !')
            alert('Tu as cliqué sur le dinosaure !') // Exemple d'action
          }
        }
      })

      // Animation
      const animate = () => {
        requestAnimationFrame(animate)
        controls.update()
        renderer.render(scene, camera)
      }

      window.addEventListener('resize', () => {
        const newWidth = container.clientWidth
        const newHeight = container.clientHeight
        renderer.setSize(newWidth, newHeight)
        camera.aspect = newWidth / newHeight
        camera.updateProjectionMatrix()
      })

      animate()
    },
  },
}
</script>

<style>
.model-container {
  width: 100vw;
  height: 100vh;
  background-color: rgb(202, 113, 113);
  overflow: hidden;
}
</style>
