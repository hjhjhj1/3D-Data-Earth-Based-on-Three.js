<template>
  <div class="w-screen h-screen bg-gradient-to-b from-slate-900 to-slate-800">
    <!-- 控制面板 -->
    <div class="absolute top-0 left-0 w-full p-6 z-10">
      <div class="bg-white/10 backdrop-blur-md rounded-lg p-4 shadow-lg">
        <div class="flex flex-wrap gap-4 items-center">
          <h1 class="text-2xl font-bold text-white">3D 数据地球</h1>
          <button
            @click="toggleRotation"
            class="px-4 py-2 bg-blue-500 hover:bg-blue-600 text-white rounded-lg transition-colors"
          >
            {{ isRotating ? '暂停旋转' : '开始旋转' }}
          </button>
          <select
            v-model="dataType"
            class="px-4 py-2 bg-gray-700 text-white rounded-lg"
          >
            <option value="heat">热力点</option>
            <option value="flight">飞行线</option>
            <option value="bar">柱状图</option>
          </select>
        </div>
      </div>
    </div>
    <!-- 3D 地球组件 -->
    <Earth :isRotating="isRotating" :dataType="dataType" />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import Earth from './components/Earth.vue'

const isRotating = ref(true)
const dataType = ref('heat')

const toggleRotation = () => {
  isRotating.value = !isRotating.value
}
</script>