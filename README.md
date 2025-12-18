# 3D-Data-Earth-Based-on-Three.js
开发基于 Three.js 的 3D 数据地球
技术
Vue 3.4+ <script setup> + Composition API
Three.js 0.160 + @tresjs/cientos
地球纹理用 Equirectangular 地图图片（public/earth.jpg）
数据用 Mock 随机生成
样式 Tailwind CSS
功能
地球可鼠标拖拽旋转、滚轮缩放（提供自动旋转开关）
热力点：根据访问量大小显示不同颜色与脉冲动画（Canvas 生成纹理贴到地球表面）
飞行线：两点间贝塞尔曲线，带拖尾动画
柱状图：国家 GDP 数据，在地球外表面垂直显示彩色柱体
顶部控制面板：暂停旋转、切换数据类型（热力/航线/柱状）
响应式：窗口 resize 自动调整相机比例
