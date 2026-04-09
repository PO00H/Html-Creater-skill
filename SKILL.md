# html-creater

Convert React/Next.js components with complex animations (Three.js, WebGL, Canvas) into standalone single-file HTML that can be opened directly in browser.

## Commands

### `/html-creater bundle <source-dir>`

**NEW**: Bundle an entire React/Vite project (multiple files) into a single standalone HTML file.

**Parameters:**
- `source-dir` (required): Path to the project directory (e.g., `.` or `src/`)
- `--output, -o`: Output filename (default: `index.html`)
- `--entry, -e`: Entry point file (default: `index.html` or `src/App.tsx`)

**Use Case:**
- Converting a complete Vite/React project to a single distributable file
- Portfolio sites that need to work offline
- Client deliverables that must open without a server

**Process:**
1. **Analyze project structure**
   - Find entry point (index.html, App.tsx, main.tsx)
   - Map all component dependencies
   - Identify CSS framework (Tailwind, custom CSS)
   - Detect animation libraries (GSAP, Framer Motion patterns)

2. **Convert React to Vanilla JS**
   - Transform JSX to template strings or DOM API
   - Convert hooks to vanilla state management
   - Replace React event system with addEventListener
   - Handle React Router (if any) with vanilla routing

3. **Process CSS**
   - Tailwind: Include CDN or extract used utilities
   - CSS Modules: Scope styles manually or convert to BEM
   - CSS-in-JS: Convert to inline styles or `<style>` blocks

4. **Handle Assets**
   - Fonts: Use Google Fonts CDN or inline
   - Images: Base64 encode small images, external URLs for large
   - SVG: Inline directly or use sprite
   - Icons: FontAwesome CDN or inline SVG

5. **Convert Animations**
   - Framer Motion → CSS animations + IntersectionObserver
   - GSAP React hooks → Direct GSAP calls
   - Custom hooks → Vanilla JS utilities

6. **Assemble Final HTML**
   - Single file with all CSS in `<style>`
   - All JS in `<script>` (or CDNs)
   - No external dependencies required

**Example:**
```bash
# Bundle entire project
/html-creater bundle . -o portfolio.html

# Custom entry point
/html-creater bundle ./project --entry src/main.tsx -o demo.html
```

**Output Structure:**
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title><!-- Project Title --></title>
  <!-- Fonts -->
  <link href="https://fonts.googleapis.com/..." rel="stylesheet">
  <!-- Tailwind (optional) -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- All CSS -->
  <style>/* ... */</style>
</head>
<body>
  <!-- All HTML structure from components -->
  
  <!-- Libraries (optional CDNs) -->
  <script src="https://unpkg.com/gsap@3.12.2/dist/gsap.min.js"></script>
  
  <!-- All JS logic converted from React -->
  <script>
    // Component logic, hooks converted to vanilla JS
    // Event handlers, animations, state management
  </script>
</body>
</html>
```

---

### `/html-creater convert <source>`

Convert a single React/Next.js component file to standalone HTML.

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

#### ES6+ to ES5

| From (ES6+) | To (ES5) |
|------------|----------|
| `class Foo {}` | `function Foo() {}` + prototype |
| `const/let` | `var` |
| Arrow functions `=>` | `function` with `bind(this)` |
| Template literals `` `...` `` | Array `join('\n')` |
| Destructuring `{a, b} = obj` | Direct property access |
| Spread `[...arr]` | `Array.prototype.concat` / `apply` |
| `Array.map/filter/reduce` | Polyfills or loops |

#### React to Vanilla JS (Bundle Mode)

| React Pattern | Vanilla JS Equivalent |
|--------------|----------------------|
| `useState()` | Object properties or data attributes |
| `useRef()` | `document.getElementById()` / `querySelector()` |
| `useEffect(() => {}, [])` | `DOMContentLoaded` event |
| `useEffect(() => {}, [dep])` | Custom event dispatching or MutationObserver |
| `onClick={handler}` | `element.addEventListener('click', handler)` |
| `className={condition ? 'a' : 'b'}` | `element.classList.toggle()` |
| `style={{prop: value}}` | `element.style.prop = value` |
| `{items.map(i => <Item />)}` | `items.map(i => html).join('')` |
| JSX conditional `{cond && <A />}` | Ternary in template literal |
| React Router | History API + hashchange |
| Context API | Global state object or custom events |

#### Animation Conversion

| React Animation | Vanilla JS |
|----------------|-----------|
| Framer Motion variants | CSS `@keyframes` |
| `animate={{x: 100}}` | CSS `transform` + `transition` |
| `whileInView` | IntersectionObserver |
| `staggerChildren` | CSS `animation-delay` with calc |
| `transition: {duration: 0.5}` | `transition-duration: 0.5s` |

**Example: Framer Motion to CSS**
```css
/* From Framer Motion */
@keyframes slideUp {
  from { opacity: 0; transform: translateY(40px); }
  to { opacity: 1; transform: translateY(0); }
}
.animate-slide-up {
  animation: slideUp 0.6s cubic-bezier(0.77, 0, 0.175, 1) forwards;
}
```

**Example: useInView hook**
```javascript
// React
const [ref, isInView] = useInView({ threshold: 0.2 });

// Vanilla JS
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.classList.add('visible');
    }
  });
}, { threshold: 0.2 });
observer.observe(element);
```

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

## Bundle Mode: Real-World Example

Converting a Vite + React + Tailwind portfolio site:

**Original Project Structure:**
```
src/
  components/
    Hero.tsx
    About.tsx
    Work.tsx
    Contact.tsx
    Navigation.tsx
    CustomCursor.tsx
  hooks/
    useInView.ts
    useSplitText.ts
  App.tsx
  index.css
tailwind.config.js
index.html
```

**Conversion Steps:**

1. **Analyze Dependencies**
   - React + ReactDOM → CDN or inline
   - Tailwind → CDN
   - GSAP → CDN (if used)
   - Custom hooks → Vanilla JS utilities

2. **Convert Components**
   ```javascript
   // Before: Hero.tsx with useState
   const [visible, setVisible] = useState(false);
   useEffect(() => setVisible(true), []);
   
   // After: Vanilla JS
   document.addEventListener('DOMContentLoaded', () => {
     document.querySelector('.hero').classList.add('visible');
   });
   ```

3. **Handle Split Text Animation**
   ```javascript
   // React component with Framer Motion
   <motion.span initial={{opacity: 0}} animate={{opacity: 1}} />
   
   // Vanilla with CSS
   .split-char {
     opacity: 0;
     animation: fadeIn 0.4s forwards;
     animation-delay: calc(var(--i) * 0.03s);
   }
   @keyframes fadeIn { to { opacity: 1; } }
   ```

4. **CSS Architecture**
   ```css
   /* Combine all styles */
   :root {
     --neon: #EAFF00;
     --black: #000000;
     --white: #FFFFFF;
   }
   /* Tailwind utilities (if not using CDN) */
   /* Component styles */
   /* Animation keyframes */
   /* Responsive media queries */
   ```

5. **Final Assembly**
   - Single HTML file ~20-50KB
   - No build step required
   - Double-click to open

---

## Resources

- **Three.js UMD**: https://unpkg.com/three@0.147.0/build/three.min.js
- **Three.js ESM**: https://unpkg.com/three@0.167.1/build/three.module.js
- **Tailwind CDN**: https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4
- **GSAP CDN**: https://unpkg.com/gsap@3.12.2/dist/gsap.min.js
- **Google Fonts**: https://fonts.google.com

## See Also

- `README.md` - Overview and introduction
- `USAGE.md` - Detailed usage instructions
- `CONVERSION_GUIDE.md` - Complete conversion reference
