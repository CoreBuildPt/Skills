---
name: 3d-printing
description: Expert technical knowledge for 3D printing file generation and design. Use this skill whenever building, reviewing, or debugging any 3D printing generator — STL/3MF export, OpenSCAD WASM, manifold geometry, wall thickness, tolerances, FDM/SLA/SLS rules, Fusion 360 export, Bambu Studio, OrcaSlicer, PrusaSlicer, Cura compatibility. Also triggers when the user mentions impressão 3D, STL, 3MF, OpenSCAD, WASM, slicer, FDM, SLA, Bambu, Prusa, Cura, espessura de parede, or any 3D generator in the Artfull project.
---

# Skill: 3D Printing — Technical Designer

You are a senior technical designer with deep expertise in 3D printing across all major workflows: parametric generation (OpenSCAD WASM), CAD export (Fusion 360), and slicer compatibility (Bambu Studio, OrcaSlicer, PrusaSlicer, Cura). Apply ALL rules below without exception when generating or reviewing 3D printing code.

---

## 1. FILE FORMAT — WHICH TO USE AND WHEN

### STL — Universal, always works
- Binary STL (not ASCII) — smaller, faster to load
- No colour, no material, no units metadata — pure triangulated geometry
- Every slicer accepts it: Bambu Studio, OrcaSlicer, PrusaSlicer, Cura, ChiTuBox
- **Chord deviation:** 0.01mm | **Angular tolerance:** 5° (not less — files become huge)
- OpenSCAD: `$fn=64` for circles, never `$fn=128+` (unnecessary, slows WASM)
- Fusion 360: Right-click component → **Save as Mesh** → Format: STL (Binary) → Refinement: **High**
  - Do NOT use File → Export (produces 3D DXF, wrong for printing)
  - Do NOT use ASCII STL — binary only

### 3MF — Better than STL, but with caveats
- Preserves: units (mm), colour (multi-material), thumbnail, part names
- Accepted by all major slicers as geometry input
- **CRITICAL INCOMPATIBILITY:** Bambu Studio saves 3MF using the **3MF Production Extension** — PrusaSlicer <2.7 and Cura cannot open these files. Prusa merged the fix in March 2024 (PrusaSlicer 2.7+)
- **Safe rule for Artfull generators:** always offer **STL download** as primary. Offer 3MF as secondary/optional
- If generating 3MF from OpenSCAD WASM: not natively supported — export STL, let slicer handle 3MF project
- Fusion 360: Right-click component → Save as Mesh → Format: 3MF (safe for all slicers as geometry)

### Format decision table for Artfull
| Use case | Format | Reason |
|----------|--------|--------|
| Primary download | STL (binary) | Universal — works on every slicer |
| Optional download | 3MF | Better for Bambu/Orca users |
| OpenSCAD WASM output | STL | WASM exports STL natively |
| Fusion 360 → slicer | STL or 3MF | Both work |
| Sharing with specific printer | STL | Avoids compatibility surprises |

---

## 2. MANIFOLD GEOMETRY — THE #1 RULE

Every STL/3MF for 3D printing MUST be **manifold (watertight)**:
- Every edge connected to EXACTLY 2 faces — no more, no less
- No gaps, holes, missing faces, T-junctions
- No two objects touching at only a single edge or vertex (they share an edge but don't intersect — non-manifold)
- Slicer cannot distinguish inside from outside on non-manifold geometry → random holes, missing layers, print failure

### OpenSCAD manifold rules (CRITICAL — apply always)
```openscad
// ❌ WRONG — two cubes share exactly one edge → non-manifold
cube([20, 20, 20]);
translate([-20, 0, 0]) cube([20, 20, 20]);

// ✅ CORRECT — force overlap by 0.01mm so they intersect
cube([20, 20, 20]);
translate([-19.99, 0, 0]) cube([20, 20, 20]);
```

```openscad
// ❌ WRONG — hole cylinder is exact same height as body → zero-thickness face
difference() {
  cube([30, 30, 10]);
  translate([15, 15, 0]) cylinder(h=10, r=3, $fn=32);
}

// ✅ CORRECT — hole extends 0.01mm beyond each face
difference() {
  cube([30, 30, 10]);
  translate([15, 15, -0.01]) cylinder(h=10.02, r=3, $fn=32);
}
```

```openscad
// ❌ WRONG — subtracted shape doesn't fully pierce the body
difference() {
  cylinder(h=10, r=15, $fn=64);
  cylinder(h=10, r=5, $fn=32);  // same height — leaves zero-thickness bottom face
}

// ✅ CORRECT — always extend subtracted shapes past both faces
difference() {
  cylinder(h=10, r=15, $fn=64);
  translate([0, 0, -0.01]) cylinder(h=10.02, r=5, $fn=32);
}
```

**Golden rule:** Any geometry used in `difference()` or `intersection()` must extend **0.01mm beyond** all faces it intersects.

---

## 3. DESIGN RULES BY TECHNOLOGY

### FDM — Filament (Bambu Lab, Prusa, Creality, Ender, Voron…)

```
Min wall thickness:         0.8mm absolute minimum | 1.5mm recommended functional
Min hole diameter (Z axis): 1.0mm vertical holes
Min hole diameter (XY):     2.0mm horizontal holes — print smaller than designed
Min feature size:           0.4mm (= 1 nozzle width — unreliable below this)
Max overhang without support: 45° from vertical
Max bridge span (no sag):   10mm reliable | 30mm with visible sag
FDM tolerance:              ±0.3mm (±0.3% for large parts)
Clearance for mating parts: 0.3mm per side (0.6mm total gap)
Text minimum height:        4mm (font) | 0.5mm depth embossed/debossed
Min wall for thin shells:   0.8mm (1 perimeter) — weaker; 1.6mm (2 perimeters) better
```

**FDM-specific compensation in generators:**
- Holes print **smaller** than designed — always add `+0.2mm` to hole diameters
- Outer dimensions print slightly **larger** — can subtract `0.05–0.1mm` for precision fits
- Horizontal holes: use **teardrop shape** (eliminates top overhang at the hole crown)
- Bottom chamfers better than fillets (chamfers print at 45° — no support needed)
- Avoid large flat horizontal surfaces that will be supported — they sag and show marks

### SLA/MSLA — Resin (Elegoo, Anycubic, Phrozen, Formlabs…)

```
Min wall thickness:  0.5mm | 0.25mm absolute minimum
Min hole diameter:   0.2mm
Accuracy:            ±0.1mm (better than FDM)
Surface finish:      Very smooth (25–50µm layers)
Drainage holes:      Min 3mm diameter for hollow parts (uncured resin trapped inside = part failure)
Hollow parts:        Max solid thickness before hollowing: 15mm (thermal stress)
Supports:            Auto supports in ChiTuBox or Lychee — always required for overhangs
```

### SLS — Powder (nylon, industrial)

```
Min wall thickness:  0.5mm XY | 1.0mm Z
Min hole diameter:   0.5mm
Escape holes needed: Min 2× 8mm holes for hollow parts (remove trapped powder)
No supports needed:  Powder bed self-supports
Accuracy:            ±0.3mm
```

---

## 4. TOLERANCE TABLE FOR ARTFULL GENERATORS

```typescript
// lib/generators/3d/tolerances.ts

export const FDM_TOLERANCES = {
  // Clearance between two printed parts that must fit together
  fitClearancePerSide: 0.3,      // mm — add to each mating surface

  // Hole compensation (holes print smaller than designed)
  holeDiameterBonus: 0.2,        // mm — add to all hole diameters

  // Text legibility
  textMinFontHeight:  4.0,       // mm
  textMinDepth:       0.5,       // mm embossed or debossed

  // Wall thickness
  wallAbsoluteMin:    0.8,       // mm — 1 perimeter, weak
  wallFunctional:     1.5,       // mm — recommended for functional parts
  wallShell:          2.5,       // mm — good for boxes/enclosures

  // Overhang
  maxOverhangDeg:     45,        // degrees from vertical — no support needed below
  maxBridgeSpan:      10,        // mm — reliable bridge without sag

  // Assembly
  snapFitClearance:   0.2,       // mm per side for snap fit features
  pressFitClearance: -0.1,       // mm per side (negative = interference fit)
} as const

export function adjustHoleDiameter(designed: number): number {
  return designed + FDM_TOLERANCES.holeDiameterBonus
}

export function adjustFitGap(dimension: number): number {
  return dimension + FDM_TOLERANCES.fitClearancePerSide * 2
}
```

---

## 5. OPENSCAD WASM — ARTFULL INTEGRATION

OpenSCAD runs in the browser via WebAssembly in a Web Worker. **Never** run it on the main thread — it blocks the UI for 5–15 seconds.

### Resolution settings — ALWAYS at top of every .scad file

```openscad
// For PREVIEW mode (fast — $fn=16 in JS params)
// For DOWNLOAD mode (quality — $fn=64 in JS params)

// Adaptive resolution — Artfull standard:
$fa = 5;    // minimum angle per arc segment (degrees)
$fs = 0.5;  // minimum segment size (mm)
$fn = 64;   // explicit circle resolution — overrides $fa/$fs when set

// DO NOT use $fn > 64 — file becomes huge, WASM takes 30+ seconds
// DO NOT use $fn < 32 for visible circles — looks faceted
// Use $fn = 16 for fast preview in browser
```

### Manifold backend — faster and more reliable

```openscad
// OpenSCAD 2024+ supports the Manifold backend via CLI flag
// In WASM: pass --enable=manifold flag when calling openscad
// Manifold is 10–50x faster than CGAL for complex boolean operations
// Results in cleaner geometry with fewer non-manifold errors
```

### Web Worker integration pattern

```typescript
// public/workers/openscad.worker.ts

import OpenSCAD from 'openscad-wasm'

let instance: Awaited<ReturnType<typeof OpenSCAD>> | null = null

// Pre-initialize to reduce first-render latency
async function init() {
  instance = await OpenSCAD({
    noInitialRun: true,
    print: () => {},       // suppress stdout
    printErr: () => {},    // suppress stderr (show errors only if export fails)
  })
}
init()

self.onmessage = async (e: MessageEvent<{
  scadCode: string
  params: Record<string, string | number | boolean>
  preview: boolean
}>) => {
  const { scadCode, params, preview } = e.data

  try {
    if (!instance) instance = await OpenSCAD({ noInitialRun: true })

    // Inject params as OpenSCAD variables before the model code
    const paramLines = Object.entries(params)
      .map(([k, v]) => typeof v === 'string' ? `${k} = "${v}";` : `${k} = ${v};`)
      .join('\n')

    // Override $fn for preview vs download
    const fnOverride = preview ? '$fn = 16;' : '$fn = 64;'

    const fullCode = `${fnOverride}\n${paramLines}\n${scadCode}`

    instance.FS.writeFile('/tmp/model.scad', fullCode)
    const exitCode = instance.callMain([
      '-o', '/tmp/output.stl',
      '--enable=manifold',   // use fast Manifold backend
      '/tmp/model.scad'
    ])

    if (exitCode !== 0) {
      self.postMessage({ error: 'OpenSCAD render failed — check geometry for non-manifold errors' })
      return
    }

    const stl = instance.FS.readFile('/tmp/output.stl')
    self.postMessage({ stl }, [stl.buffer])

  } catch (err) {
    self.postMessage({ error: String(err) })
  }
}
```

### React component — standard Artfull pattern

```typescript
// components/generators/Generator3D.tsx
'use client'
import { useState, useEffect, useRef, useCallback } from 'react'

type Status = 'idle' | 'generating' | 'done' | 'error'

export function Generator3D({ scadTemplate, defaultParams, generatorName }: Props) {
  const [params, setParams]   = useState(defaultParams)
  const [status, setStatus]   = useState<Status>('idle')
  const [progress, setProgress] = useState(0)
  const [stlBuffer, setStlBuffer] = useState<Uint8Array | null>(null)
  const workerRef = useRef<Worker | null>(null)
  const timerRef  = useRef<ReturnType<typeof setInterval> | null>(null)

  useEffect(() => {
    workerRef.current = new Worker('/workers/openscad.worker.js')
    workerRef.current.onmessage = (e) => {
      clearInterval(timerRef.current!)
      if (e.data.error) {
        setStatus('error')
        return
      }
      setStlBuffer(e.data.stl)
      setProgress(100)
      setStatus('done')
    }
    return () => workerRef.current?.terminate()
  }, [])

  const generate = useCallback((forDownload = false) => {
    const errors = validateParams(params)
    if (errors.length > 0) return   // show errors in UI

    setStatus('generating')
    setProgress(0)

    // Simulate progress — OpenSCAD doesn't report %
    timerRef.current = setInterval(() => {
      setProgress(p => Math.min(p + 8, 88))
    }, 700)

    workerRef.current!.postMessage({
      scadCode: scadTemplate,
      params,
      preview: !forDownload,
    })
  }, [params, scadTemplate])

  return (
    <div className="generator-3d">
      {/* Parameter inputs here */}

      {status === 'generating' && (
        <div className="generating-state">
          <p>A gerar o teu modelo 3D…</p>
          <p className="hint">(pode demorar até 15 segundos)</p>
          <progress value={progress} max={100} />
          <button onClick={() => { workerRef.current?.terminate(); setStatus('idle') }}>
            Cancelar
          </button>
        </div>
      )}

      {status === 'error' && (
        <p className="error">Erro ao gerar. Verifica os parâmetros e tenta de novo.</p>
      )}

      {status === 'done' && stlBuffer && (
        <div className="done-state">
          <STLPreview buffer={stlBuffer} />
          <button onClick={() => downloadSTL(stlBuffer, generatorName, params)}>
            Descarregar STL
          </button>
        </div>
      )}
    </div>
  )
}
```

---

## 6. SCAD TEMPLATES — ARTFULL GENERATORS

### Golden rules for ALL templates:
1. Always use `difference()` with **+0.01mm overlap** on holes
2. Text always `linear_extrude` + `text()` — never try to do 3D text another way
3. All params injected by JS — never hardcode dimensions
4. `$fn` controlled by JS injection — never set in template
5. `center=true` on all primitives unless the base must be at Z=0
6. Comments explain every module

### Template A — 3D Text / Name plate
```openscad
// Artfull — Gerador de Texto 3D
// Params injected by JS: text_content, font_size, text_depth, base, base_margin, base_thickness

module text_shape() {
  text(text_content,
       size    = font_size,
       font    = "Liberation Sans:style=Bold",
       halign  = "center",
       valign  = "center",
       $fn     = $fn);
}

if (base) {
  // Base plate — slightly larger than text bounding box
  linear_extrude(height = base_thickness)
    offset(r = base_margin)
      text_shape();
}

// Text extrusion on top of base
translate([0, 0, base ? base_thickness : 0])
  linear_extrude(height = text_depth)
    text_shape();
```

### Template B — Parametric box with lid
```openscad
// Artfull — Caixa Paramétrica com Tampa
// Params: box_w, box_d, box_h, wall, lid_h, tolerance

module box_body() {
  difference() {
    cube([box_w, box_d, box_h]);
    // Interior cavity — wall offset on all sides, open top
    translate([wall, wall, wall])
      cube([box_w - wall*2, box_d - wall*2, box_h]);
  }
}

module lid() {
  lip = lid_h - wall;
  union() {
    // Top plate
    cube([box_w, box_d, wall]);
    // Lip that fits inside box with tolerance
    translate([tolerance, tolerance, -lip])
      difference() {
        cube([box_w - tolerance*2, box_d - tolerance*2, lip]);
        translate([wall, wall, -0.01])
          cube([box_w - (wall + tolerance)*2, box_d - (wall + tolerance)*2, lip + 0.02]);
      }
  }
}

// Render both parts side by side for printing
box_body();
translate([box_w + 10, 0, 0]) lid();
```

### Template C — Phone/tablet stand
```openscad
// Artfull — Suporte Paramétrico
// Params: device_w, device_h (optional), base_angle, stand_d, wall_t, cable_hole

base_w = device_w + wall_t * 2 + 4;

module stand() {
  difference() {
    union() {
      // Base
      cube([base_w, stand_d, wall_t]);
      // Back support — angled
      rotate([90 - base_angle, 0, 0])
        translate([0, 0, 0])
          cube([base_w, wall_t, stand_d / sin(base_angle)]);
      // Front lip to hold device
      translate([0, 0, wall_t])
        cube([base_w, wall_t * 2, 8]);
    }
    // Cable hole at base (if enabled)
    if (cable_hole)
      translate([base_w/2, -0.01, wall_t/2])
        rotate([-90, 0, 0])
          cylinder(h = wall_t * 2 + 0.02, r = 5, $fn=32);
  }
}

stand();
```

### Template D — Keychain / Tag
```openscad
// Artfull — Chaveiro Personalizado
// Params: tag_text, tag_w, tag_h, tag_thick, text_depth, ring_d

module tag_body() {
  hull() {
    translate([ring_d/2 + 2, ring_d/2 + 2, 0])
      cylinder(h=tag_thick, r=ring_d/2 + 2, $fn=32);
    translate([tag_w - 3, 3, 0])
      cylinder(h=tag_thick, r=3, $fn=32);
    translate([tag_w - 3, tag_h - 3, 0])
      cylinder(h=tag_thick, r=3, $fn=32);
    translate([ring_d/2 + 2, tag_h - 3, 0])
      cylinder(h=tag_thick, r=3, $fn=32);
  }
}

difference() {
  tag_body();
  // Ring hole
  translate([ring_d/2 + 2, ring_d/2 + 2, -0.01])
    cylinder(h=tag_thick + 0.02, r=ring_d/2, $fn=32);
  // Embossed text
  translate([tag_w/2, tag_h/2, tag_thick - text_depth])
    linear_extrude(height=text_depth + 0.01)
      text(tag_text, size=6, font="Liberation Sans:style=Bold",
           halign="center", valign="center");
}
```

---

## 7. FUSION 360 → SLICER EXPORT WORKFLOW

### Method 1: Save as Mesh (recommended for Artfull docs)
```
1. Design workspace
2. Browser (left panel) → Right-click on Component or Body
3. Select "Save as Mesh"
4. Format: STL (Binary)              ← always binary, not ASCII
5. Refinement: High                  ← for curved surfaces
   OR Custom: Deviation 0.01mm, Angle 5°
6. Units: Millimeters                ← CRITICAL — not inches
7. Click OK → choose location → Save
```

**Pitfall:** If model appears 25.4× too large in slicer → units were in inches, not mm. Fix in Fusion: Document Settings → Change Units to mm, re-export.

### Method 2: 3D Print Utility (sends directly to slicer)
```
File → 3D Print → select body/component
→ Refinement: High
→ "Send to 3D Print Utility" → choose Bambu Studio / PrusaSlicer
```

### What NOT to do in Fusion 360
- ❌ File → Export → DXF (creates 3D DXF — useless for printing)
- ❌ Export as ASCII STL (larger, slower)
- ❌ Export assembly as single file with multiple bodies in separate positions (origins mismatch)
- ❌ Export with construction geometry visible (verify with Inspect → Show/Hide bodies)

---

## 8. SLICER COMPATIBILITY MATRIX

| Feature | Bambu Studio | OrcaSlicer | PrusaSlicer | Cura |
|---------|-------------|------------|-------------|------|
| STL import | ✅ | ✅ | ✅ | ✅ |
| 3MF import (geometry) | ✅ | ✅ | ✅ (2.7+) | ✅ |
| Bambu 3MF project | ✅ | ✅ | ✅ (2.7+) | ❌ |
| Tree supports | ✅ | ✅ Smart | ✅ Organic | ✅ |
| Multi-colour AMS | ✅ native | ✅ | via MMU | via Palette |
| File accepts STEP | ✅ | ✅ | ✅ | ❌ |

### Bambu Studio specifics
- Supports: `.3mf .stl .stp .step .amf .obj`
- 3MF saved by Bambu uses **Production Extension** — not compatible with old PrusaSlicer/Cura
- AMS multi-colour: paint on model in Bambu Studio, not pre-assigned in file
- Recommended for Artfull users: offer STL — user imports into Bambu, assigns filament colour in AMS settings
- `.gcode.3mf` = sliced plate file for direct printing — NOT a geometry file

### OrcaSlicer specifics
- Fork of Bambu Studio — same 3MF compatibility
- Smart Supports: Advanced tab → auto tree supports (12% less material than regular)
- Precise Wall: compensates for oval extrusion path overlap (improves dimensional accuracy)
- Accepts same file types as Bambu Studio

### PrusaSlicer specifics
- Supports: `.stl .obj .amf .3mf .step`
- v2.7+ opens Bambu 3MF after patch merged March 2024
- Paint-on supports: right-click model → Support Enforcer/Blocker
- Variable layer height: great for Artfull models with both detail and bulk sections
- Recommended settings for Artfull STLs: 0.2mm layer, 15% gyroid infill, Organic supports

### Cura specifics
- Supports: `.stl .obj .x3d .3mf .bmp .gif .jpg .png`
- Cannot open Bambu 3MF project files
- Per-model settings: right-click → Per Model Settings (powerful for multi-part prints)
- Tree supports available as standard feature
- Wall thickness note: Cura assumes **rectangular** extrusion path; PrusaSlicer/Orca assume **oval** → same wall count = slightly different actual thickness (important for precision fits)

---

## 9. VALIDATION — RUN BEFORE GENERATING

```typescript
// lib/generators/3d/validate.ts

interface Params3D {
  width?:        number
  height?:       number
  depth?:        number
  wallThickness?: number
  textContent?:  string
  fontSize?:     number
  holeDiameter?: number
}

export function validate3DParams(p: Params3D): string[] {
  const errors: string[] = []

  // Dimensional limits
  if (p.width  && p.width  > 300) errors.push('Largura superior a 300mm: verifica se cabe na tua impressora')
  if (p.height && p.height > 400) errors.push('Altura superior a 400mm: requer impressora grande (ex: Bambu P1S)')
  if (p.depth  && p.depth  > 300) errors.push('Profundidade superior a 300mm: verifica capacidade da impressora')

  // Wall thickness
  if (p.wallThickness !== undefined) {
    if (p.wallThickness < 0.4)  errors.push('Espessura de parede inferior a 0.4mm: impossível de imprimir em FDM')
    if (p.wallThickness < 0.8)  errors.push('Espessura de parede inferior a 0.8mm: resultado muito frágil')
    if (p.wallThickness < 1.5)  errors.push('Recomendado: espessura ≥ 1.5mm para peças funcionais')
  }

  // Text
  if (p.textContent !== undefined && p.textContent.trim().length === 0)
    errors.push('Texto não pode estar vazio')
  if (p.fontSize !== undefined && p.fontSize < 4)
    errors.push('Tamanho mínimo de fonte: 4mm para texto legível em FDM')

  // Holes
  if (p.holeDiameter !== undefined && p.holeDiameter < 2)
    errors.push('Diâmetro de furo mínimo recomendado: 2mm (furos pequenos fecham em FDM)')

  return errors
}
```

---

## 10. THREE.JS PREVIEW — STANDARD IMPLEMENTATION

```typescript
// components/generators/STLPreview.tsx
'use client'
import { useEffect, useRef } from 'react'
import * as THREE from 'three'
import { STLLoader } from 'three/examples/jsm/loaders/STLLoader'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

export function STLPreview({ buffer }: { buffer: Uint8Array }) {
  const mountRef = useRef<HTMLDivElement>(null)

  useEffect(() => {
    if (!buffer || !mountRef.current) return
    const el = mountRef.current
    const w = el.clientWidth  || 400
    const h = el.clientHeight || 400

    // Scene
    const scene    = new THREE.Scene()
    scene.background = new THREE.Color(0x1A1A1A)  // --bg-surface Artfull

    const camera   = new THREE.PerspectiveCamera(45, w / h, 0.1, 2000)
    const renderer = new THREE.WebGLRenderer({ antialias: true })
    renderer.setSize(w, h)
    renderer.setPixelRatio(window.devicePixelRatio)
    el.appendChild(renderer.domElement)

    const controls = new OrbitControls(camera, renderer.domElement)
    controls.enableDamping = true
    controls.dampingFactor = 0.05

    // Load STL
    const geometry = new STLLoader().parse(buffer.buffer)
    geometry.computeBoundingBox()
    geometry.computeVertexNormals()

    // Centre model at origin
    const center = new THREE.Vector3()
    geometry.boundingBox!.getCenter(center)
    geometry.translate(-center.x, -center.y, -center.z)

    const material = new THREE.MeshPhongMaterial({
      color:    0xE8402A,   // --accent Artfull
      specular: 0x333333,
      shininess: 50,
    })
    const mesh = new THREE.Mesh(geometry, material)
    scene.add(mesh)

    // Lighting
    scene.add(new THREE.AmbientLight(0x404040, 1.5))
    const key  = new THREE.DirectionalLight(0xffffff, 1.2)
    key.position.set(1, 2, 3)
    scene.add(key)
    const fill = new THREE.DirectionalLight(0x8080ff, 0.4)
    fill.position.set(-2, 0, -1)
    scene.add(fill)

    // Auto-fit camera to model
    const box  = new THREE.Box3().setFromObject(mesh)
    const size = box.getSize(new THREE.Vector3()).length()
    camera.position.set(size * 0.6, size * 0.4, size * 1.2)
    camera.lookAt(0, 0, 0)
    controls.update()

    let rafId: number
    const animate = () => {
      rafId = requestAnimationFrame(animate)
      controls.update()
      renderer.render(scene, camera)
    }
    animate()

    return () => {
      cancelAnimationFrame(rafId)
      controls.dispose()
      renderer.dispose()
      if (el.contains(renderer.domElement)) el.removeChild(renderer.domElement)
    }
  }, [buffer])

  return <div ref={mountRef} style={{ width: '100%', aspectRatio: '1', borderRadius: 8 }} />
}
```

---

## 11. STL DOWNLOAD — CORRECT IMPLEMENTATION

```typescript
// lib/generators/3d/download.ts

export function downloadSTL(buffer: Uint8Array, generator: string, params: Record<string, unknown>) {
  const filename = buildFilename(generator, params)
  const blob = new Blob([buffer], { type: 'application/octet-stream' })
  const url  = URL.createObjectURL(blob)
  const a    = document.createElement('a')
  a.href     = url
  a.download = filename
  document.body.appendChild(a)
  a.click()
  document.body.removeChild(a)
  setTimeout(() => URL.revokeObjectURL(url), 1000)
}

function buildFilename(generator: string, params: Record<string, unknown>): string {
  // artfull-texto3d-OLA_MUNDO-20mm.stl
  const key = Object.values(params)
    .slice(0, 2)
    .map(v => String(v).replace(/[^a-zA-Z0-9]/g, '_').slice(0, 20))
    .join('-')
  return `artfull-${generator}-${key}.stl`
}
```

---

## 12. COMMON ERRORS — DIAGNOSIS AND FIX

| Problem | Cause | Fix |
|---------|-------|-----|
| "Object isn't a valid 2-manifold" in OpenSCAD | Two shapes touch at single edge/vertex | Add 0.01mm overlap on all difference() cutters |
| STL imports at 25.4× wrong size | Fusion exported in inches not mm | Fusion: Document Settings → set units to mm, re-export |
| Holes too tight after printing | FDM holes print smaller than designed | Add +0.2mm to all hole diameters in generator |
| Parts don't snap together | No clearance tolerance | Add 0.3mm per mating side in generator |
| Text not legible | Font too small or depth too shallow | Min 4mm height, 0.5mm depth embossed |
| Bambu 3MF won't open in PrusaSlicer | Production Extension incompatibility | Use STL as primary format; 3MF as optional |
| Thin walls disappear in slicer | Wall < nozzle width (< 0.4mm) | Enforce min 0.8mm in validation |
| WASM crashes browser tab | Running on main thread | Always run OpenSCAD in Web Worker |
| Preview loads forever | $fn too high | Use $fn=16 for preview, $fn=64 for download |
| STL file too large (>50MB) | $fn=128+ or extremely complex | Cap $fn at 64; split complex models |
| Overhang fails mid-print | >45° unsupported | Redesign with chamfers or add support markers in SCAD |
| Slicer shows inverted/inside-out model | Normals flipped | In OpenSCAD: use positive extrusions; check difference() order |
