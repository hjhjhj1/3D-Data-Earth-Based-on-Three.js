<template>
  <div class="w-screen h-screen bg-gradient-to-b from-slate-900 to-slate-800">
    <!-- 顶部控制面板 -->
    <div class="absolute top-4 left-1/2 transform -translate-x-1/2 z-10">
      <div class="bg-white/90 backdrop-blur-sm rounded-lg shadow-xl p-4 flex items-center gap-4">
        <button
          @click="toggleRotation"
          class="px-4 py-2 bg-blue-500 hover:bg-blue-600 text-white rounded-lg transition-colors"
        >
          {{ isRotating ? '暂停旋转' : '开始旋转' }}
        </button>
        
        <div class="flex items-center gap-2">
          <span class="text-sm font-medium text-gray-700">数据类型：</span>
          <select
            v-model="selectedDataType"
            class="px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
          >
            <option value="heatmap">热力点</option>
            <option value="flight">飞行线</option>
            <option value="bar">柱状图</option>
          </select>
        </div>
      </div>
    </div>

    <!-- 3D 地球组件 -->
    <Earth
      ref="earthRef"
      :is-rotating="isRotating"
      :selected-data-type="selectedDataType"
    />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import Earth from './components/Earth.vue'

const isRotating = ref(true)
const selectedDataType = ref('heatmap')
const earthRef = ref(null)

const toggleRotation = () => {
  isRotating.value = !isRotating.value
}
</script>
