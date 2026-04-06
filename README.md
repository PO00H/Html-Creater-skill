# HTML Creater Skill

<div align="center">

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/poh/html-creater-skill?style=flat)](https://github.com/poh/html-creater-skill/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/poh/html-creater-skill?style=flat)](https://github.com/poh/html-creater-skill/network)

**Language / 语言**: [English](#english) | [简体中文](#简体中文)

</div>

---

<a name="english"></a>
## English

### Tech Stack

![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)
![Three.js](https://img.shields.io/badge/Three.js-000000?style=for-the-badge&logo=three.js&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)
![WebGL](https://img.shields.io/badge/WebGL-990000?style=for-the-badge&logo=webgl&logoColor=white)
![TailwindCSS](https://img.shields.io/badge/Tailwind_CSS-38B2AC?style=for-the-badge&logo=tailwind-css&logoColor=white)

### Overview

Convert React/Next.js components with complex animations (Three.js, WebGL, Canvas) into standalone single-file HTML that can be opened directly in browser.

**Source**: React + Three.js + npm dependencies + build tools  
**Output**: Single `.html` file, double-click to open

### When to Use

- Converting portfolio/demo sites to standalone files
- Creating offline-capable interactive presentations
- Embedding complex animations in third-party platforms
- Sharing WebGL experiments without build setup

### Key Capabilities

| Feature | Supported |
|---------|-----------|
| Three.js WebGL scenes | Full support (r147 UMD) |
| React Hooks lifecycle | Mapped to native JS |
| ES6+ syntax | Auto-downgrade to ES5 |
| ES Modules | Convert to UMD/IIFE |
| Tailwind CSS | Via CDN |
| Custom shaders | Inline string conversion |
| FBO/RenderTargets | Full fluid simulation support |

### Quick Start

```bash
# Use the skill
/html-creater convert <source-file>

# Example
/html-creater convert components/LiquidEther.tsx --output portfolio.html
```

### Commands

| Command | Description |
|---------|-------------|
| `/html-creater convert <source>` | Convert React component to standalone HTML |
| `/html-creater analyze <source>` | Analyze conversion requirements |
| `/html-creater template <name>` | Generate starter template |

### Templates Included

- `three-fluid` - Navier-Stokes fluid simulation
- `three-basic` - Basic Three.js scene setup
- `canvas-2d` - Canvas 2D animation template

---

<a name="简体中文"></a>
## 简体中文

### 技术栈

![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)
![Three.js](https://img.shields.io/badge/Three.js-000000?style=for-the-badge&logo=three.js&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)
![WebGL](https://img.shields.io/badge/WebGL-990000?style=for-the-badge&logo=webgl&logoColor=white)
![TailwindCSS](https://img.shields.io/badge/Tailwind_CSS-38B2AC?style=for-the-badge&logo=tailwind-css&logoColor=white)

### 项目简介

将带有复杂动画的 React/Next.js 组件（Three.js、WebGL、Canvas）转换为可直接在浏览器中打开的独立单文件 HTML。

**输入**：React + Three.js + npm 依赖 + 构建工具  
**输出**：单个 `.html` 文件，双击即可打开

### 适用场景

- 将作品集/演示站点转换为独立文件
- 创建支持离线的交互式演示
- 在第三方平台嵌入复杂动画
- 无需构建配置即可分享 WebGL 实验

### 核心功能

| 功能 | 支持情况 |
|---------|-----------|
| Three.js WebGL 场景 | 完全支持 (r147 UMD) |
| React Hooks 生命周期 | 映射为原生 JS |
| ES6+ 语法 | 自动降级至 ES5 |
| ES 模块 | 转换为 UMD/IIFE |
| Tailwind CSS | 通过 CDN |
| 自定义着色器 | 内联字符串转换 |
| FBO/渲染目标 | 完整流体模拟支持 |

### 快速开始

```bash
# 使用技能
/html-creater convert <源文件>

# 示例
/html-creater convert components/LiquidEther.tsx --output portfolio.html
```

### 命令列表

| 命令 | 说明 |
|---------|-------------|
| `/html-creater convert <source>` | 将 React 组件转换为独立 HTML |
| `/html-creater analyze <source>` | 分析转换需求 |
| `/html-creater template <name>` | 生成起始模板 |

### 包含模板

- `three-fluid` - Navier-Stokes 流体模拟
- `three-basic` - 基础 Three.js 场景设置
- `canvas-2d` - Canvas 2D 动画模板

---

## Output Variants / 输出版本

| Variant | Can Double-Click | Three.js Version | Use Case |
|---------|-----------------|------------------|----------|
| `standalone` | Yes | r147 (UMD) | Distribution, offline / 分发、离线使用 |
| `esm` | No (needs server) | r167+ (ESM) | Development, modern browsers / 开发、现代浏览器 |

## Architecture / 架构

```
React/Next.js Component
        │
        ▼
┌─────────────────────────────────┐
│  1. Dependency Analysis         │
│     - Check npm packages        │
│     - Find UMD alternatives     │
├─────────────────────────────────┤
│  2. Syntax Transformation       │
│     - ES6 class → ES5 prototype │
│     - const/let → var           │
│     - Arrow → function.bind()   │
├─────────────────────────────────┤
│  3. Module System Conversion    │
│     - ES Module → UMD global    │
│     - import → <script src>     │
├─────────────────────────────────┤
│  4. Lifecycle Mapping           │
│     - useEffect → events        │
│     - useRef → variables        │
│     - cleanup → beforeunload    │
├─────────────────────────────────┤
│  5. Resource Inlining           │
│     - CSS → <style>             │
│     - Shaders → string arrays   │
│     - Assets → base64           │
└─────────────────────────────────┘
        │
        ▼
Single HTML File (UMD/ESM)
```

## Resources / 相关资源

- Three.js UMD: https://unpkg.com/three@0.147.0/build/three.min.js
- Tailwind CDN: https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4

## License / 许可证

MIT License - see [LICENSE](LICENSE) file for details.
