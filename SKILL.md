# html-creater

Convert React/Next.js components with complex animations (Three.js, WebGL, Canvas) into standalone single-file HTML that can be opened directly in browser.

## Commands

### `/html-creater convert <source>`

Convert a React/Next.js component file to standalone HTML.

**Parameters:**
- `source` (required): Path to the source React component (e.g., `components/LiquidEther.tsx`)
- `--output, -o`: Output filename (default: `index-standalone.html`)
- `--variant, -v`: `standalone` (default, double-clickable) or `esm` (needs server)
- `--title, -t`: HTML page title

**Examples:**
```bash
# Basic conversion - double-clickable output
/html-creater convert components/LiquidEther.tsx

# Custom output name
/html-creater convert app/page.tsx -o portfolio.html -t "My Portfolio"

# ES Module variant (requires HTTP server)
/html-creater convert src/Scene.tsx -v esm -o scene.html
```

**Process:**
1. Analyze source file for dependencies
2. Convert ES6+ syntax to ES5
3. Transform React hooks to native JS
4. Inline shaders (template strings → array join)
5. Generate single HTML file with all resources

---

### `/html-creater analyze <source>`

Analyze a component and report conversion requirements.

**Parameters:**
- `source` (required): Path to source file

**Output includes:**
- Detected dependencies and their UMD availability
- ES6+ features requiring conversion
- Estimated complexity score
- Recommended variant (standalone vs esm)

---

### `/html-creater template <name>`

Generate a starter template.

**Templates:**
- `three-fluid`: Navier-Stokes fluid simulation base
- `three-basic`: Minimal Three.js scene setup
- `canvas-2d`: 2D canvas particle system

**Example:**
```bash
/html-creater template three-fluid -o my-fluid.html
```

---

## Conversion Methodology

### Phase 1: Dependency Analysis

Identify and resolve:
- **Three.js**: Use UMD r147 for standalone, ESM r167+ for modern
- **Tailwind**: CDN `browser@4`
- **Other libs**: Find UMD builds or inline

### Phase 2: Syntax Transformation

| From (ES6+) | To (ES5) |
|------------|----------|
| `class Foo {}` | `function Foo() {}` + prototype |
| `const/let` | `var` |
| Arrow functions `=>` | `function` with `bind(this)` |
| Template literals `` `...` `` | Array `join('\n')` |
| `useState` | Object properties |
| `useRef` | Variables |
| `useEffect` | IIFE / event listeners |

### Phase 3: Module System

**Standalone (file:// compatible):**
```html
<script src="https://unpkg.com/three@0.147.0/build/three.min.js"></script>
<script>
  // Global THREE
  var scene = new THREE.Scene();
</script>
```

**ESM (requires server):**
```html
<script type="importmap">
{"imports": {"three": "https://unpkg.com/three@0.167.1/build/three.module.js"}}
</script>
<script type="module">
  import * as THREE from 'three';
</script>
```

### Phase 4: Resource Inlining

- CSS → `<style>` tag
- Shaders → Array join strings
- Small assets → base64 data URIs

---

## Key Decisions

### When to use Standalone (UMD)
- Need to double-click open
- Distributing to clients
- Offline usage
- Embedding in email/presentations

### When to use ESM
- Development workflow
- Latest Three.js features needed
- Tree-shaking important
- OK to run HTTP server

---

## Common Issues

### "THREE is not defined"
- Check script loaded before your code
- Use r147 or earlier for UMD

### "Cannot use import statement"
- Using ES Module without `type="module"`
- Or trying ES Module from file://

### Shader compilation errors
- RawShaderMaterial needs explicit attributes
- Template strings not properly converted

### "this is undefined"
- Lost binding in callback
- Use `var self = this` pattern

---

## Templates

### three-fluid
Complete Navier-Stokes fluid simulation with:
- FBO ping-pong rendering
- Advection, divergence, pressure solvers
- Mouse interaction
- Color palette mapping

### three-basic
Minimal Three.js setup with:
- Scene, camera, renderer
- Orbit-like mouse controls
- Animation loop
- Responsive resize

### canvas-2d
2D particle system with:
- Canvas API
- Particle class
- Connection lines
- Mouse interaction

---

## Resources

- **Three.js UMD**: https://unpkg.com/three@0.147.0/build/three.min.js
- **Three.js ESM**: https://unpkg.com/three@0.167.1/build/three.module.js
- **Tailwind CDN**: https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4

## See Also

- `README.md` - Overview and introduction
- `USAGE.md` - Detailed usage instructions
- `CONVERSION_GUIDE.md` - Complete conversion reference
