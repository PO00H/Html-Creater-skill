# HTML Creater Usage Guide

## Commands

### `/html-creater convert <source>`

Convert a React/Next.js component to standalone HTML.

**Options:**
- `--output, -o`: Output filename (default: `index-standalone.html`)
- `--variant, -v`: `standalone` (default) or `esm`
- `--title, -t`: HTML page title
- `--cdn`: Use CDN for Three.js instead of inline

**Examples:**

```bash
# Basic conversion (standalone, double-clickable)
/html-creater convert components/LiquidEther.tsx

# Specify output name
/html-creater convert app/page.tsx -o portfolio.html

# ES Module variant (requires server)
/html-creater convert src/Scene.tsx -v esm -o scene.html

# With custom title
/html-creater convert components/Hero.tsx -t "My Portfolio" -o hero.html
```

### `/html-creater analyze <source>`

Analyze a component and report:
- Dependencies that need UMD alternatives
- ES6+ syntax that needs conversion
- Estimated output size

### `/html-creater template <name>`

Generate a starter template:
- `three-fluid` - Full fluid simulation base
- `three-basic` - Minimal Three.js setup
- `canvas-2d` - 2D canvas animation

## Step-by-Step Conversion Workflow

### Step 1: Analyze Dependencies

```bash
/html-creater analyze components/MyComponent.tsx
```

**Checklist:**
- [ ] Is Three.js used? → Use `three@0.147.0` (UMD)
- [ ] Other npm packages? → Find UMD builds or inline
- [ ] CSS framework? → Use CDN links
- [ ] External assets? → Convert to base64 or CDN

### Step 2: Extract Core Logic

**From React component, identify:**

| React Concept | What to Extract |
|--------------|-----------------|
| `useEffect(() => {...}, [])` | Initialization code |
| `useEffect(() => {...}, [dep])` | Update logic → Event handlers |
| `useRef` | Variables that persist |
| `useState` | State object + manual updates |
| JSX return | DOM structure to recreate |

### Step 3: Syntax Conversion

**ES6 → ES5 Transformations:**

```javascript
// BEFORE (React/ES6)
class Simulation {
  constructor(options) {
    this.options = { ...options };
  }
  
  update(dt) {
    this.options.dt = dt;
  }
}

const sim = new Simulation({ dt: 0.1 });

// AFTER (Standalone HTML)
function Simulation(options) {
  this.options = options || {};
}

Simulation.prototype.update = function(dt) {
  this.options.dt = dt;
};

var sim = new Simulation({ dt: 0.1 });
```

**Arrow Functions:**

```javascript
// BEFORE
window.addEventListener('resize', () => {
  this.resize();
});

// AFTER
var self = this;
window.addEventListener('resize', function() {
  self.resize();
});
```

**Template Strings (Shaders):**

```javascript
// BEFORE
const fragShader = `
  precision highp float;
  varying vec2 vUv;
  void main() {
    gl_FragColor = vec4(vUv, 0.0, 1.0);
  }
`;

// AFTER
var fragShader = [
  'precision highp float;',
  'varying vec2 vUv;',
  'void main() {',
  '  gl_FragColor = vec4(vUv, 0.0, 1.0);',
  '}'
].join('\n');
```

### Step 4: Module System Choice

#### Option A: Standalone (Double-Clickable)

```html
<!DOCTYPE html>
<html>
<head>
  <!-- UMD Libraries -->
  <script src="https://unpkg.com/three@0.147.0/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
</head>
<body>
  <script>
    // Global THREE available
    var scene = new THREE.Scene();
    
    // Your code wrapped in IIFE
    (function() {
      // All component logic here
    })();
  </script>
</body>
</html>
```

#### Option B: ES Module (Modern)

```html
<!DOCTYPE html>
<html>
<head>
  <script type="importmap">
  {
    "imports": {
      "three": "https://unpkg.com/three@0.167.1/build/three.module.js"
    }
  }
  </script>
</head>
<body>
  <script type="module">
    import * as THREE from 'three';
    // Your code here
  </script>
</body>
</html>
```

### Step 5: Lifecycle Mapping

| React Hook | Native Equivalent |
|-----------|-------------------|
| `useEffect(() => {...}, [])` | `DOMContentLoaded` event |
| `useEffect(() => {...}, [dep])` | Custom event or RAF loop |
| `useEffect(() => () => cleanup, [])` | `beforeunload` event |
| `useRef` | Top-level variable |
| `useState` | Object property + render trigger |

### Step 6: Event System

```javascript
// React
const [mouse, setMouse] = useState({ x: 0, y: 0 });
useEffect(() => {
  const handler = (e) => setMouse({ x: e.clientX, y: e.clientY });
  window.addEventListener('mousemove', handler);
  return () => window.removeEventListener('mousemove', handler);
}, []);

// Native JS
var Mouse = {
  coords: { x: 0, y: 0 },
  init: function() {
    this._handler = this.onMove.bind(this);
    window.addEventListener('mousemove', this._handler, { passive: true });
  },
  onMove: function(e) {
    this.coords.x = e.clientX;
    this.coords.y = e.clientY;
  },
  destroy: function() {
    window.removeEventListener('mousemove', this._handler);
  }
};
Mouse.init();
```

### Step 7: Inline Resources

**CSS:**
```html
<style>
  /* Original CSS content here */
  #canvas { position: fixed; top: 0; left: 0; }
</style>
```

**Small Images (Base64):**
```html
<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUg..." />
```

## Common Patterns

### Three.js WebGL Scene

```javascript
// Setup
var canvas = document.getElementById('canvas');
var renderer = new THREE.WebGLRenderer({ canvas: canvas, antialias: true });
var scene = new THREE.Scene();
var camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);

// Resize
window.addEventListener('resize', function() {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
}, { passive: true });

// Animation
function animate() {
  requestAnimationFrame(animate);
  renderer.render(scene, camera);
}
animate();
```

### RawShaderMaterial with FBO

```javascript
// Shader strings
var vertexShader = [
  'attribute vec3 position;',
  'void main() {',
  '  gl_Position = vec4(position, 1.0);',
  '}'
].join('\n');

// FBO setup
var fbo = new THREE.WebGLRenderTarget(width, height, {
  type: THREE.FloatType,
  minFilter: THREE.LinearFilter,
  magFilter: THREE.LinearFilter
});

// Material
var material = new THREE.RawShaderMaterial({
  vertexShader: vertexShader,
  fragmentShader: fragmentShader,
  uniforms: {
    uTexture: { value: fbo.texture }
  }
});
```

## Troubleshooting

### "THREE is not defined"
- Check UMD script loaded before your code
- Ensure using `three@0.147.0` or earlier for UMD

### "Cannot use import statement outside a module"
- For standalone: Use `<script>` not `type="module"`
- For ESM: Add `type="module"` and Import Map

### "Access to script at 'file://...' from origin 'null' has been blocked"
- ES Modules cannot run from `file://`
- Use standalone UMD version instead

### Shader compilation errors
- Check attribute declarations for RawShaderMaterial
- Ensure WebGL 1.0 syntax (`varying`, `gl_FragColor`)

## Best Practices

1. **Always wrap in IIFE** to avoid global namespace pollution
2. **Use `{ passive: true }`** for scroll/touch events
3. **Clean up event listeners** in `beforeunload`
4. **Check for WebGL support** before initializing
5. **Handle resize** for responsive canvas
6. **Pause animation** when tab is hidden (`visibilitychange`)
