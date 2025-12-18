<template>
  <div ref="containerRef" class="w-full h-full"></div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'

const props = defineProps({
  isRotating: {
    type: Boolean,
    default: true
  },
  dataType: {
    type: String,
    default: 'heat'
  }
})

const containerRef = ref(null)
let scene = null
let camera = null
let renderer = null
let earth = null
let controls = null
let heatPoints = []
let flightLines = []
let barCharts = []
let animationId = null

// 生成随机数据
const generateMockData = () => {
  const heatData = []
  const flightData = []
  const barData = []

  // 生成热力点数据
  for (let i = 0; i < 50; i++) {
    heatData.push({
      lat: (Math.random() - 0.5) * 180,
      lng: (Math.random() - 0.5) * 360,
      value: Math.random() * 1000
    })
  }

  // 生成飞行线数据
  for (let i = 0; i < 30; i++) {
    flightData.push({
      from: {
        lat: (Math.random() - 0.5) * 180,
        lng: (Math.random() - 0.5) * 360
      },
      to: {
        lat: (Math.random() - 0.5) * 180,
        lng: (Math.random() - 0.5) * 360
      }
    })
  }

  // 生成柱状图数据
  for (let i = 0; i < 40; i++) {
    barData.push({
      lat: (Math.random() - 0.5) * 180,
      lng: (Math.random() - 0.5) * 360,
      value: Math.random() * 500
    })
  }

  return { heatData, flightData, barData }
}

// 经纬度转三维坐标
const latLngToVector3 = (lat, lng, radius = 1) => {
  const phi = (90 - lat) * (Math.PI / 180)
  const theta = (lng + 180) * (Math.PI / 180)

  const x = -(radius * Math.sin(phi) * Math.cos(theta))
  const z = (radius * Math.sin(phi) * Math.sin(theta))
  const y = (radius * Math.cos(phi))

  return new THREE.Vector3(x, y, z)
}

// 创建地球
const createEarth = () => {
  const geometry = new THREE.SphereGeometry(1, 64, 64)
  const textureLoader = new THREE.TextureLoader()
  const texture = textureLoader.load('/earth.svg')
  const material = new THREE.MeshPhongMaterial({ map: texture })
  earth = new THREE.Mesh(geometry, material)
  scene.add(earth)
}

// 创建热力点
const createHeatPoints = (data) => {
  heatPoints.forEach(point => {
    earth.remove(point.mesh)
    earth.remove(point.pulseMesh)
  })
  heatPoints = []

  data.forEach(item => {
    const position = latLngToVector3(item.lat, item.lng, 1.01)

    // 创建主点
    const canvas = document.createElement('canvas')
    canvas.width = 64
    canvas.height = 64
    const ctx = canvas.getContext('2d')
    const gradient = ctx.createRadialGradient(32, 32, 0, 32, 32, 32)
    gradient.addColorStop(0, 'rgba(255, 255, 0, 1)')
    gradient.addColorStop(0.5, 'rgba(255, 165, 0, 0.8)')
    gradient.addColorStop(1, 'rgba(255, 0, 0, 0)')
    ctx.fillStyle = gradient
    ctx.fillRect(0, 0, 64, 64)
    const texture = new THREE.CanvasTexture(canvas)
    const spriteMaterial = new THREE.SpriteMaterial({ map: texture })
    const sprite = new THREE.Sprite(spriteMaterial)
    sprite.position.copy(position)
    sprite.scale.setScalar(0.1 * (item.value / 1000))

    // 创建脉冲效果
    const pulseCanvas = document.createElement('canvas')
    pulseCanvas.width = 64
    pulseCanvas.height = 64
    const pulseCtx = pulseCanvas.getContext('2d')
    const pulseGradient = pulseCtx.createRadialGradient(32, 32, 0, 32, 32, 32)
    pulseGradient.addColorStop(0, 'rgba(255, 255, 0, 0)')
    pulseGradient.addColorStop(0.5, 'rgba(255, 165, 0, 0.5)')
    pulseGradient.addColorStop(1, 'rgba(255, 0, 0, 0)')
    pulseCtx.fillStyle = pulseGradient
    pulseCtx.fillRect(0, 0, 64, 64)
    const pulseTexture = new THREE.CanvasTexture(pulseCanvas)
    const pulseSpriteMaterial = new THREE.SpriteMaterial({ map: pulseTexture })
    const pulseSprite = new THREE.Sprite(pulseSpriteMaterial)
    pulseSprite.position.copy(position)
    pulseSprite.scale.setScalar(0.1 * (item.value / 1000))

    earth.add(sprite)
    earth.add(pulseSprite)

    heatPoints.push({ mesh: sprite, pulseMesh: pulseSprite, value: item.value, scale: 0.1 * (item.value / 1000) })
  })
}

// 创建飞行线
const createFlightLines = (data) => {
  flightLines.forEach(line => {
    earth.remove(line)
  })
  flightLines = []

  data.forEach(item => {
    const from = latLngToVector3(item.from.lat, item.from.lng, 1.1)
    const to = latLngToVector3(item.to.lat, item.to.lng, 1.1)
    const mid = from.clone().add(to).multiplyScalar(0.5)
    mid.multiplyScalar(1.5)

    const curve = new THREE.QuadraticBezierCurve3(from, mid, to)
    const points = curve.getPoints(50)
    const geometry = new THREE.BufferGeometry().setFromPoints(points)
    const material = new THREE.LineBasicMaterial({ color: 0x00ffff, transparent: true, opacity: 0.6 })
    const line = new THREE.Line(geometry, material)

    earth.add(line)
    flightLines.push(line)
  })
}

// 创建柱状图
const createBarCharts = (data) => {
  barCharts.forEach(bar => {
    earth.remove(bar)
  })
  barCharts = []

  data.forEach(item => {
    const position = latLngToVector3(item.lat, item.lng, 1.0)
    const direction = position.clone().normalize()
    const height = item.value / 1000

    const geometry = new THREE.BoxGeometry(0.05, height, 0.05)
    const material = new THREE.MeshPhongMaterial({ color: new THREE.Color().setHSL(Math.random(), 0.7, 0.5) })
    const bar = new THREE.Mesh(geometry, material)

    bar.position.copy(position.clone().add(direction.multiplyScalar(height / 2)))
    bar.lookAt(0, 0, 0)

    earth.add(bar)
    barCharts.push(bar)
  })
}

// 初始化场景
const initScene = () => {
  // 创建场景
  scene = new THREE.Scene()
  scene.background = new THREE.Color(0x000510)

  // 创建相机
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.set(0, 0, 3)

  // 创建渲染器
  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.shadowMap.enabled = true
  containerRef.value.appendChild(renderer.domElement)

  // 添加控制器
  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.dampingFactor = 0.05

  // 添加光源
  const ambientLight = new THREE.AmbientLight(0x404040, 0.5)
  scene.add(ambientLight)
  const directionalLight = new THREE.DirectionalLight(0xffffff, 1)
  directionalLight.position.set(5, 3, 5)
  scene.add(directionalLight)

  // 创建地球
  createEarth()

  // 生成并加载数据
  const mockData = generateMockData()
  createHeatPoints(mockData.heatData)
  createFlightLines(mockData.flightData)
  createBarCharts(mockData.barData)

  // 监听窗口大小变化
  window.addEventListener('resize', onWindowResize)
}

// 窗口大小变化处理
const onWindowResize = () => {
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth, window.innerHeight)
}

// 动画循环
const animate = () => {
  animationId = requestAnimationFrame(animate)

  // 更新控制器
  controls.update()

  // 地球自动旋转
  if (earth && props.isRotating) {
    earth.rotation.y += 0.002
  }

  // 热力点脉冲动画
  heatPoints.forEach(point => {
    const time = Date.now() * 0.002
    const scale = point.scale * (1 + Math.sin(time) * 0.3)
    point.pulseMesh.scale.setScalar(scale)
  })

  // 渲染场景
  renderer.render(scene, camera)
}

// 监听数据类型变化
watch(() => props.dataType, (newType) => {
  // 显示对应的数据类型
  heatPoints.forEach(point => {
    point.mesh.visible = newType === 'heat'
    point.pulseMesh.visible = newType === 'heat'
  })
  flightLines.forEach(line => {
    line.visible = newType === 'flight'
  })
  barCharts.forEach(bar => {
    bar.visible = newType === 'bar'
  })
})

// 挂载时初始化
onMounted(() => {
  initScene()
  animate()
})

// 卸载时清理
onUnmounted(() => {
  if (animationId) {
    cancelAnimationFrame(animationId)
  }
  window.removeEventListener('resize', onWindowResize)
  if (renderer) {
    renderer.dispose()
  }
})
</script>