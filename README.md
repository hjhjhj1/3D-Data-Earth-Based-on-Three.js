# 3D 数据地球可视化

基于 Three.js 的交互式 3D 地球数据可视化项目，支持多种数据展示方式和丰富的交互功能。

## ✨ 特性

- 🎨 **现代化技术栈**: Vue 3.4+ Composition API + Three.js 0.160
- 🌍 **真实地球纹理**: 使用 Equirectangular 地图纹理
- 🎮 **丰富交互**: 鼠标拖拽旋转、滚轮缩放、自动旋转
- 📊 **多种数据可视化**:
  - 热力点：访问量大小可视化，带脉冲动画
  - 飞行线：两点间贝塞尔曲线，带拖尾效果
  - 柱状图：国家 GDP 数据，彩色柱体展示
- 🎛️ **控制面板**: 旋转开关、数据类型切换
- 📱 **响应式设计**: 窗口 resize 自动适配
- 🎨 **美观 UI**: Tailwind CSS 打造现代化界面

## 🚀 快速开始

### 安装依赖

```bash
npm install
```

### 启动开发服务器

```bash
npm run dev
```

### 构建生产版本

```bash
npm run build
```

### 预览生产构建

```bash
npm run preview
```

## 📁 项目结构

```
.
├── public/                 # 静态资源
│   └── earth.jpg          # 地球纹理图片
├── src/
│   ├── components/
│   │   └── Earth.vue     # 3D 地球核心组件
│   ├── App.vue            # 主应用组件
│   ├── main.js            # 应用入口
│   └── style.css          # 全局样式
├── index.html             # HTML 模板
├── vite.config.js         # Vite 配置
├── tailwind.config.js     # Tailwind CSS 配置
├── postcss.config.js      # PostCSS 配置
└── package.json           # 项目配置
```

## 🎮 使用说明

### 交互控制

- **鼠标拖拽**: 旋转地球
- **滚轮**: 缩放视图
- **控制面板**:
  - "暂停旋转" / "开始旋转": 控制地球自动旋转
  - "数据类型": 切换热力点、飞行线、柱状图

### 数据可视化

1. **热力点**: 显示访问量数据，颜色从黄色到红色渐变，带脉冲动画
2. **飞行线**: 两点间的飞行航线，使用贝塞尔曲线，带拖尾效果
3. **柱状图**: 国家 GDP 数据，在地球表面垂直显示彩色柱体

## 🛠️ 技术栈

- **Vue 3.4+**: 渐进式 JavaScript 框架
- **Three.js 0.160**: 3D 图形库
- **@tresjs/cientos**: Three.js 的 Vue 组件库
- **Tailwind CSS**: 实用优先的 CSS 框架
- **Vite**: 下一代前端构建工具

## 📝 注意事项

- 确保在 `public` 目录下放置 `earth.jpg` 地球纹理图片
- 项目使用 Mock 数据，可根据实际需求替换为真实数据
- 建议在现代浏览器中运行，以获得最佳性能

## 📄 许可证

MIT License
