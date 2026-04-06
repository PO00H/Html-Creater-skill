# HTML Creater Skill

Convert React/Next.js components with complex animations (Three.js, WebGL, Canvas) into standalone single-file HTML that can be opened directly in browser.

## Purpose

Transforms modern React-based interactive experiences into portable, serverless HTML files:
- **Source**: React + Three.js + npm dependencies + build tools
- **Output**: Single `.html` file, double-click to open

## When to Use

- Converting portfolio/demo sites to standalone files
- Creating offline-capable interactive presentations
- Embedding complex animations in third-party platforms
- Sharing WebGL experiments without build setup

## Key Capabilities

| Feature | Supported |
|---------|-----------|
| Three.js WebGL scenes | ✅ Full support (r147 UMD) |
| React Hooks lifecycle | ✅ Mapped to native JS |
| ES6+ syntax | ✅ Auto-downgrade to ES5 |
| ES Modules | ✅ Convert to UMD/IIFE |
| Tailwind CSS | ✅ Via CDN |
| Custom shaders | ✅ Inline string conversion |
| FBO/RenderTargets | ✅ Full fluid simulation support |

## Quick Start

```bash
# Use the skill
/html-creater convert <source-file>

# Example
/html-creater convert components/LiquidEther.tsx --output portfolio.html
```

## Output Variants

| Variant | Can Double-Click | Three.js Version | Use Case |
|---------|-----------------|------------------|----------|
| `standalone` | ✅ Yes | r147 (UMD) | Distribution, offline |
| `esm` | ❌ No (needs server) | r167+ (ESM) | Development, modern browsers |

## Architecture

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

## Templates Included

- `three-fluid` - Navier-Stokes fluid simulation (Liquid Ether)
- `three-basic` - Basic Three.js scene setup
- `canvas-2d` - Canvas 2D animation template
- `react-component` - Generic React to HTML converter

## Related

- Three.js UMD: https://unpkg.com/three@0.147.0/build/three.min.js
- Tailwind CDN: https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4
