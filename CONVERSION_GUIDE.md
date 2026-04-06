# HTML Creater: Conversion Guide

## Overview

This guide documents the complete process of converting React/Next.js components with complex animations to standalone HTML files.

## Architecture Transformation

```
┌─────────────────────────────────────────────────────────────┐
│                    SOURCE (React/Next.js)                    │
├─────────────────────────────────────────────────────────────┤
│  Build System: Webpack/Vite                                  │
│  Dependencies: npm packages                                  │
│  Module System: ES Modules                                   │
│  Syntax: ES6+ (class, const, arrow functions)                │
│  Styling: CSS Modules / Tailwind / Styled Components         │
│  3D Engine: Three.js (npm: three@0.167.0+)                   │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼ Convert
┌─────────────────────────────────────────────────────────────┐
│              TARGET (Standalone HTML)                        │
├─────────────────────────────────────────────────────────────┤
│  Build System: None (browser native)                         │
│  Dependencies: CDN links or inline                           │
│  Module System: UMD (global variables) or Import Map         │
│  Syntax: ES5 (function, var, prototypes)                     │
│  Styling: Inline <style> or CDN                              │
│  3D Engine: Three.js (UMD: three@0.147.0)                    │
└─────────────────────────────────────────────────────────────┘
```

## Step-by-Step Conversion Process

### Step 1: Dependency Analysis

Identify all dependencies in the source component:

```javascript
// React component imports
import * as THREE from 'three';           // → Three.js
import { useEffect, useRef } from 'react'; // → Remove (native equivalent)
import './styles.css';                     // → Inline
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'; // → Find UMD
```

**Dependency Resolution Matrix:**

| Package | npm Version | UMD/CDN Alternative | Notes |
|---------|-------------|---------------------|-------|
| three | 0.167.0 | 0.147.0 (last UMD) | ES Module only after r160 |
| @react-three/fiber | latest | N/A | Must inline or convert to vanilla |
| gsap | latest | gsap.min.js (CDN) | Full UMD support |
| tailwindcss | npm | browser@4 (CDN) | Runtime compiler |

### Step 2: Module System Selection

#### Option A: UMD (Standalone, Double-Clickable)

```html
<script src="https://unpkg.com/three@0.147.0/build/three.min.js"></script>
<script>
  // THREE is global
  var scene = new THREE.Scene();
</script>
```

**Pros:**
- Works with `file://` protocol
- No build step needed
- Simple to understand

**Cons:**
- Limited to older Three.js (r147)
- No tree-shaking
- Global namespace pollution

#### Option B: ES Module with Import Map

```html
<script type="importmap">
{
  "imports": {
    "three": "https://unpkg.com/three@0.167.1/build/three.module.js"
  }
}
</script>
<script type="module">
  import * as THREE from 'three';
</script>
```

**Pros:**
- Latest Three.js features
- Proper module scoping
- Future-proof

**Cons:**
- Requires HTTP server (CORS restriction)
- More complex setup

### Step 3: Syntax Conversion (ES6 → ES5)

#### Class Declaration

```javascript
// ES6 (React)
class Simulation {
  constructor(options) {
    this.options = { ...options };
    this.initialized = false;
  }

  update(dt) {
    this.options.dt = dt;
    this.initialized = true;
  }

  static createDefault() {
    return new Simulation({ dt: 0.1 });
  }
}

// ES5 (HTML)
function Simulation(options) {
  this.options = options || {};
  this.initialized = false;
}

Simulation.prototype.update = function(dt) {
  this.options.dt = dt;
  this.initialized = true;
};

Simulation.createDefault = function() {
  return new Simulation({ dt: 0.1 });
};
```

#### Variable Declaration

```javascript
// ES6
const PI = 3.14159;
let count = 0;
for (const item of items) { }

// ES5
var PI = 3.14159;
var count = 0;
for (var i = 0; i < items.length; i++) {
  var item = items[i];
}
```

#### Arrow Functions

```javascript
// ES6
const handleResize = () => {
  this.resize();
};

items.map(item => item.id);

// ES5
var self = this;
var handleResize = function() {
  self.resize();
};

items.map(function(item) {
  return item.id;
});
```

#### Template Literals (Critical for Shaders)

```javascript
// ES6 (Template string with newlines)
const vertexShader = `
  attribute vec3 position;
  varying vec2 vUv;
  
  void main() {
    vUv = uv;
    gl_Position = vec4(position, 1.0);
  }
`;

// ES5 (Array join)
var vertexShader = [
  'attribute vec3 position;',
  'varying vec2 vUv;',
  '',
  'void main() {',
  '  vUv = uv;',
  '  gl_Position = vec4(position, 1.0);',
  '}'
].join('\n');

// Alternative (escaped string - NOT recommended)
var vertexShader = 'attribute vec3 position;\nvarying vec2 vUv;\n\nvoid main() {\n  vUv = uv;\n  gl_Position = vec4(position, 1.0);\n}';
```

### Step 4: React Hooks → Native JavaScript

#### useState

```javascript
// React
const [count, setCount] = useState(0);
const [mouse, setMouse] = useState({ x: 0, y: 0 });

setCount(count + 1);
setMouse({ x: e.clientX, y: e.clientY });

// Native JS
var state = {
  count: 0,
  mouse: { x: 0, y: 0 }
};

state.count++;
state.mouse.x = e.clientX;
state.mouse.y = e.clientY;
```

#### useRef

```javascript
// React
const canvasRef = useRef(null);
const rendererRef = useRef(null);

useEffect(() => {
  rendererRef.current = new THREE.WebGLRenderer({
    canvas: canvasRef.current
  });
}, []);

// Native JS
var canvas = document.getElementById('canvas');
var renderer = null;

function init() {
  renderer = new THREE.WebGLRenderer({ canvas: canvas });
}
```

#### useEffect Lifecycle

```javascript
// React
useEffect(() => {
  // Mount
  const handler = () => console.log('resize');
  window.addEventListener('resize', handler);
  
  // Animation loop
  const animate = () => {
    requestAnimationFrame(animate);
    render();
  };
  animate();
  
  // Cleanup
  return () => {
    window.removeEventListener('resize', handler);
  };
}, []);

// Native JS (IIFE pattern)
(function() {
  // Mount
  var handler = function() { console.log('resize'); };
  window.addEventListener('resize', handler);
  
  // Animation loop
  function animate() {
    requestAnimationFrame(animate);
    render();
  }
  animate();
  
  // Cleanup (optional, page unload)
  window.addEventListener('beforeunload', function() {
    window.removeEventListener('resize', handler);
  });
})();
```

### Step 5: Three.js Specific Conversions

#### RawShaderMaterial

```javascript
// Both versions same, but ES5 shader strings
var material = new THREE.RawShaderMaterial({
  vertexShader: vertexShader,     // From array join
  fragmentShader: fragmentShader, // From array join
  uniforms: {
    uTime: { value: 0 },
    uResolution: { value: new THREE.Vector2() }
  }
});
```

#### FBO (Frame Buffer Object)

```javascript
// Same in both, just syntax differences
var fbo = new THREE.WebGLRenderTarget(width, height, {
  type: THREE.FloatType,
  minFilter: THREE.LinearFilter,
  magFilter: THREE.LinearFilter,
  wrapS: THREE.ClampToEdgeWrapping,
  wrapT: THREE.ClampToEdgeWrapping
});
```

#### Shader Pass Structure

```javascript
// ES6 Class
class ShaderPass {
  constructor(props) {
    this.props = props;
    this.scene = new THREE.Scene();
    this.camera = new THREE.Camera();
    this.material = new THREE.RawShaderMaterial(props.material);
    this.plane = new THREE.Mesh(
      new THREE.PlaneGeometry(2, 2),
      this.material
    );
    this.scene.add(this.plane);
  }
  
  update() {
    renderer.render(this.scene, this.camera);
  }
}

// ES5 Prototype
function ShaderPass(props) {
  this.props = props;
  this.scene = new THREE.Scene();
  this.camera = new THREE.Camera();
  this.material = new THREE.RawShaderMaterial(props.material);
  this.plane = new THREE.Mesh(
    new THREE.PlaneGeometry(2, 2),
    this.material
  );
  this.scene.add(this.plane);
}

ShaderPass.prototype.update = function() {
  renderer.render(this.scene, this.camera);
};
```

### Step 6: Event Handling

```javascript
// React
const [mouse, setMouse] = useState({ x: 0, y: 0 });

useEffect(() => {
  const handleMouseMove = (e) => {
    setMouse({ x: e.clientX, y: e.clientY });
  };
  
  window.addEventListener('mousemove', handleMouseMove);
  return () => window.removeEventListener('mousemove', handleMouseMove);
}, []);

// Native JS (namespace pattern)
var Input = {
  mouse: { x: 0, y: 0 },
  
  init: function() {
    this._onMove = this.onMove.bind(this);
    window.addEventListener('mousemove', this._onMove, { passive: true });
  },
  
  onMove: function(e) {
    this.mouse.x = e.clientX;
    this.mouse.y = e.clientY;
  },
  
  destroy: function() {
    window.removeEventListener('mousemove', this._onMove);
  }
};

Input.init();
```

### Step 7: Resource Inlining

#### CSS

```html
<!-- External (React) -->
import './styles.css';

<!-- Inline (HTML) -->
<style>
  body { margin: 0; overflow: hidden; }
  canvas { display: block; }
</style>
```

#### Images (Base64 for small assets)

```html
<!-- External -->
<img src="/assets/logo.png" />

<!-- Inline -->
<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUg..." />
```

## Common Pitfalls

### 1. CORS Error with ES Modules

**Error:** `Access to script at 'file://...' from origin 'null' has been blocked`

**Solution:** Use UMD version for `file://` access, or run HTTP server

### 2. Shader Compilation Error

**Error:** `Vertex shader is not compiled`

**Cause:** RawShaderMaterial requires explicit attribute declarations

**Solution:**
```glsl
// Must include in vertex shader
attribute vec3 position;
attribute vec2 uv;
```

### 3. This Binding Loss

**Error:** `Cannot read property 'x' of undefined`

**Cause:** Arrow functions preserve `this`, regular functions don't

**Solution:**
```javascript
var self = this;
callback(function() {
  self.method(); // NOT this.method()
});
```

### 4. Three.js Version Mismatch

**Error:** `THREE.XXX is not a constructor`

**Cause:** Using features from newer Three.js in UMD (r147)

**Solution:** Check Three.js changelog for version-specific features

## Quick Reference Card

```
┌────────────────────────────────────────────────────┐
│ ES6 → ES5 Cheat Sheet                              │
├────────────────────────────────────────────────────┤
│ class {}           → function() + prototype        │
│ const/let          → var                           │
│ => {}              → function() {}                 │
│ `template`         → ['array'].join('\n')          │
│ {...spread}        → Object.assign()               │
│ [a, b] = arr       → var a=arr[0], b=arr[1]        │
│ import {x} from '' → Global variable (UMD)         │
│ useState()         → Object property               │
│ useRef()           → Variable                      │
│ useEffect(()=>{},[])→ IIFE / DOMContentLoaded      │
└────────────────────────────────────────────────────┘
```

## Validation Checklist

Before considering conversion complete:

- [ ] File opens correctly with double-click
- [ ] No console errors
- [ ] Canvas displays correctly
- [ ] Mouse/touch interactions work
- [ ] Resize handling works
- [ ] Animation loop runs smoothly
- [ ] Memory usage stable (no leaks)
- [ ] Mobile/touch devices supported

## Advanced Topics

### Worker Threads

For heavy computations, use Web Workers:

```javascript
// Inline worker using Blob
var workerScript = [
  'self.onmessage = function(e) {',
  '  // Heavy computation',
  '  self.postMessage(result);',
  '}'
].join('\n');

var blob = new Blob([workerScript], { type: 'application/javascript' });
var worker = new Worker(URL.createObjectURL(blob));
```

### Local Storage for Settings

```javascript
// Save
localStorage.setItem('settings', JSON.stringify(settings));

// Load
var settings = JSON.parse(localStorage.getItem('settings') || '{}');
```

### Performance Monitoring

```javascript
var stats = {
  fps: 0,
  frameCount: 0,
  lastTime: performance.now()
};

function updateStats() {
  stats.frameCount++;
  var time = performance.now();
  if (time >= stats.lastTime + 1000) {
    stats.fps = (stats.frameCount * 1000) / (time - stats.lastTime);
    stats.frameCount = 0;
    stats.lastTime = time;
    console.log('FPS:', stats.fps);
  }
}
```

## Resources

- **Three.js UMD:** https://unpkg.com/three@0.147.0/build/three.min.js
- **Tailwind CDN:** https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4
- **ES5 Compatibility:** https://kangax.github.io/compat-table/es5/
- **WebGL Stats:** https://webglstats.com/
