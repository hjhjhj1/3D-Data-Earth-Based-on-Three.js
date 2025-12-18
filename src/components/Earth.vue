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
  selectedDataType: {
    type: String,
    default: 'heatmap'
  }
})

const containerRef = ref(null)
let scene = null
let camera = null
let renderer = null
let controls = null
let earthMesh = null
let rotationId = null
let heatmapPoints = []
let flightPaths = []
let barCharts = []

// 生成 Mock 数据
const generateMockData = () => {
  // 热力点数据
  const heatmapData = []
  for (let i = 0; i < 50; i++) {
    heatmapData.push({
      lat: (Math.random() - 0.5) * 180,
      lng: (Math.random() - 0.5) * 360,
      value: Math.random() * 100
    })
  }

  // 飞行线数据
  const flightData = []
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

  // 柱状图数据
  const barData = []
  for (let i = 0; i < 40; i++) {
    barData.push({
      lat: (Math.random() - 0.5) * 180,
      lng: (Math.random() - 0.5) * 360,
      value: Math.random() * 5 + 1
    })
  }

  return { heatmapData, flightData, barData }
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
  const material = new THREE.MeshStandardMaterial({
    map: texture,
    roughness: 0.8,
    metalness: 0.2
  })
  earthMesh = new THREE.Group()
  const earthSphere = new THREE.Mesh(geometry, material)
  earthMesh.add(earthSphere)
  scene.add(earthMesh)
}

// 创建热力点
const createHeatmapPoints = (data) => {
  data.forEach(point => {
    const position = latLngToVector3(point.lat, point.lng, 1.01)

    // 创建 Canvas 纹理用于脉冲动画
    const canvas = document.createElement('canvas')
    canvas.width = 64
    canvas.height = 64
    const ctx = canvas.getContext('2d')

    // 绘制脉冲点
    const gradient = ctx.createRadialGradient(32, 32, 0, 32, 32, 32)
    gradient.addColorStop(0, 'rgba(255, 0, 0, 1)')
    gradient.addColorStop(0.5, 'rgba(255, 165, 0, 0.8)')
    gradient.addColorStop(1, 'rgba(255, 255, 0, 0)')

    ctx.fillStyle = gradient
    ctx.fillRect(0, 0, 64, 64)

    const texture = new THREE.CanvasTexture(canvas)
    const spriteMaterial = new THREE.SpriteMaterial({ map: texture })
    const sprite = new THREE.Sprite(spriteMaterial)
    sprite.position.copy(position)
    sprite.scale.setScalar(0.1 * (point.value / 100) + 0.05)

    earthMesh.add(sprite)
    heatmapPoints.push(sprite)
  })
}

// 创建飞行线
const createFlightPaths = (data) => {
  data.forEach(path => {
    const from = latLngToVector3(path.from.lat, path.from.lng, 1.1)
    const to = latLngToVector3(path.to.lat, path.to.lng, 1.1)

    // 创建贝塞尔曲线路径
    const mid = new THREE.Vector3()
    mid.addVectors(from, to).multiplyScalar(0.5)
    mid.normalize().multiplyScalar(1.5)

    const curve = new THREE.QuadraticBezierCurve3(from, mid, to)
    const points = curve.getPoints(50)
    const geometry = new THREE.BufferGeometry().setFromPoints(points)

    const material = new THREE.LineBasicMaterial({
      color: new THREE.Color().setHSL(Math.random(), 0.8, 0.6),
      transparent: true,
      opacity: 0.7
    })

    const line = new THREE.Line(geometry, material)
    earthMesh.add(line)
    flightPaths.push(line)
  })
}

// 创建柱状图
const createBarCharts = (data) => {
  data.forEach(bar => {
    const position = latLngToVector3(bar.lat, bar.lng, 1)
    const direction = position.clone().normalize()

    const height = bar.value * 0.4 // 调整柱状图高度
    const geometry = new THREE.CylinderGeometry(0.01, 0.02, height, 12) // 渐变柱体形状
    const material = new THREE.MeshStandardMaterial({
      color: new THREE.Color().setHSL(0.3 + bar.value * 0.05, 0.8, 0.5),
      emissive: new THREE.Color().setHSL(0.3 + bar.value * 0.05, 0.8, 0.2),
      shininess: 100
    })

    const cylinder = new THREE.Mesh(geometry, material)

    // 定位并旋转柱体
    cylinder.position.copy(position.clone().add(direction.multiplyScalar(height / 2)))
    cylinder.quaternion.setFromUnitVectors(new THREE.Vector3(0, 1, 0), direction)

    earthMesh.add(cylinder)
    barCharts.push(cylinder)

    // 添加柱状图顶部装饰
    const topGeometry = new THREE.SphereGeometry(0.015, 12, 12)
    const topMaterial = new THREE.MeshStandardMaterial({
      color: new THREE.Color().setHSL(0.1 + bar.value * 0.05, 0.9, 0.6),
      emissive: new THREE.Color().setHSL(0.1 + bar.value * 0.05, 0.9, 0.3),
      transparent: true,
      opacity: 0.9
    })
    const top = new THREE.Mesh(topGeometry, topMaterial)
    const topPosition = position.clone().add(direction.multiplyScalar(height * 0.55))
    top.position.copy(topPosition)
    earthMesh.add(top)
    barCharts.push(top)
  })
}

// 清除所有数据可视化
const clearDataVisualization = () => {
  heatmapPoints.forEach(sprite => earthMesh.remove(sprite))
  flightPaths.forEach(line => earthMesh.remove(line))
  barCharts.forEach(bar => earthMesh.remove(bar))
  heatmapPoints = []
  flightPaths = []
  barCharts = []
}

// 初始化场景
const initScene = () => {
  scene = new THREE.Scene()
  scene.background = new THREE.Color(0x0a0e27)

  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.set(0, 0, 3)

  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.shadowMap.enabled = true
  containerRef.value.appendChild(renderer.domElement)

  // 添加光照
  const ambientLight = new THREE.AmbientLight(0x404040, 0.5)
  scene.add(ambientLight)

  const directionalLight = new THREE.DirectionalLight(0xffffff, 1)
  directionalLight.position.set(5, 3, 5)
  directionalLight.castShadow = true
  scene.add(directionalLight)

  const pointLight = new THREE.PointLight(0xffffff, 0.5)
  pointLight.position.set(-5, -3, -5)
  scene.add(pointLight)

  // 控制器
  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.dampingFactor = 0.05
  controls.minDistance = 1.5
  controls.maxDistance = 10

  // 创建地球
  createEarth()

  // 生成并添加数据
  const mockData = generateMockData()
  createHeatmapPoints(mockData.heatmapData)
  createFlightPaths(mockData.flightData)
  createBarCharts(mockData.barData)

  // 隐藏默认显示的数据
  flightPaths.forEach(line => line.visible = false)
  barCharts.forEach(bar => bar.visible = false)

  // 开始动画循环
  animate()
}

// 动画循环
const animate = () => {
  requestAnimationFrame(animate)

  if (earthMesh && props.isRotating) {
    earthMesh.rotation.y += 0.002
  }

  // 脉冲动画
  heatmapPoints.forEach((sprite, index) => {
    const scale = 0.1 + Math.sin(Date.now() * 0.003 + index) * 0.05
    sprite.scale.setScalar(scale)
  })

  controls.update()
  renderer.render(scene, camera)
}

// 响应式处理
const handleResize = () => {
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth, window.innerHeight)
}

// 监听数据类型变化
watch(() => props.selectedDataType, (newType) => {
  heatmapPoints.forEach(sprite => sprite.visible = newType === 'heatmap')
  flightPaths.forEach(line => line.visible = newType === 'flight')
  barCharts.forEach(bar => bar.visible = newType === 'bar')
})

// 生命周期
onMounted(() => {
  initScene()
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
  if (renderer) {
    renderer.dispose()
  }
})
</script>
