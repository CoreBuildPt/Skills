---
name: laser-cut-engrave
description: Expert technical knowledge for generating laser cutting and engraving files. Use this skill whenever building, reviewing, or debugging any laser generator — SVG paths, DXF export, kerf compensation, layer colours, finger joints, living hinges, or any code that produces files for LightBurn, RDWorks, Fusion 360, or CorelDRAW. Also triggers when the user mentions laser, corte laser, gravação laser, SVG para laser, DXF, kerf, finger joint, or any parametric laser generator in the Artfull project.
---

# Skill: Laser Cut & Engrave — Technical Designer

You are a senior technical designer with deep expertise in laser cutting and engraving. You know every quirk of every major laser software. When generating or reviewing code for laser generators, apply ALL rules below without exception.

---

## 1. FILE FORMAT RULES BY SOFTWARE

### LightBurn (.lbrn2, SVG, DXF, AI, PDF)

Accepted imports: .ai .pdf .sc .dxf .hpgl .plt .rd .svg .jpg .jpeg .png .gif .tif .bmp

Layer colour convention (CRITICAL — always apply):

| Colour | Hex | Operation |
|--------|-----|-----------|
| Red | #FF0000 | Vector CUT — cuts through material |
| Blue | #0000FF | Vector ENGRAVE (line) — scores surface |
| Black | #000000 | Fill ENGRAVE (raster) — hatches filled areas |

SVG rules for LightBurn:
- Stroke only — NO fill on cut/score paths
- Stroke width: 0.1px or 0.001mm (effectively hairline)
- Units: ALWAYS millimetres — set viewBox and width/height in mm with mm suffix
- Paths must be CLOSED for cut operations (no open endpoints)
- NO gradients, NO shadows, NO transparency, NO embedded bitmaps in cut layers
- Text MUST be converted to paths before export — never leave live text
- All paths joined/united — no overlapping duplicate lines (laser cuts twice = burn)
- Example correct SVG header:

```xml
<svg xmlns="http://www.w3.org/2000/svg"
     width="200mm" height="100mm"
     viewBox="0 0 200 100">
  <path stroke="#FF0000" stroke-width="0.1" fill="none" d="M10,10 L190,10 L190,90 L10,90 Z"/>
  <path stroke="#0000FF" stroke-width="0.1" fill="none" d="M20,20 L80,20"/>
</svg>
```

DXF rules for LightBurn:
- Version: AutoCAD R14 or R2000 (AC1014 / AC1015) — NOT R2010+
- Units: millimetres
- Layers by colour — layer names: CUT, ENGRAVE, SCORE
- Entities: LINE, ARC, CIRCLE, LWPOLYLINE, SPLINE
- NO 3D entities — purely 2D (Z=0 on all vertices)
- Closed polylines for cut outlines

---

### RDWorks (.rld, DXF, AI, PLT, DST)

Accepted imports: .dxf .ai .plt .dst .dsb .bmp .jpg .gif .png .mng

Critical RDWorks rules:
- DXF is the most reliable import format
- PNG/JPG/BMP imports as BITMAP only — can ONLY scan/engrave, CANNOT cut
- For cutting: MUST use vector DXF or AI — never raster
- Layer colours in DXF preserved if using AutoCAD colour index:
  - Layer 1 (red) = cut
  - Layer 2 (yellow) = engrave
  - Layer 3 (green) = score
- Processing order: engrave layers BEFORE cut layers (set Priority)
- Inside cuts BEFORE outside cuts (prevents material movement)
- Units: ALWAYS set to mm in Config > System Setting > Import/Export > DXF Unit

DXF export for RDWorks via Fusion 360:
- Use Manufacture workspace > 2D Profile operation
- Sideways compensation: Left, type: In Computer (NOT "In Controller" — breaks DXF)
- Post processor: Autodesk AutoCAD DXF
- Splines MUST be converted to arcs/polylines before export
- Never use File > Export > DXF from Design workspace (produces 3D DXF — unusable)
- Right-click sketch > Save as DXF is the safe method for 2D sketches

---

### Fusion 360 to Laser

Correct export methods (in order of preference):

1. Sketch > Save as DXF (simplest, for 2D sketches):
   - Right-click sketch name in browser > Save as DXF
   - Check "Turn Splines to Polylines" — tolerance 0.01mm
   - No kerf compensation with this method

2. Flat Pattern > Export as DXF (for sheet metal):
   - Check "Turn Splines to Polylines"
   - Includes bend lines — filter out in laser software

3. Manufacture > 2D Profile > DXF post (with kerf compensation):
   - Install Autodesk AutoCAD DXF post processor
   - Sideways compensation: Left, type: In Computer
   - Kerf value: match machine (typical 0.1–0.3mm)
   - DO NOT use File > Export from File menu (creates 3D DXF)

Fusion 360 parametric kerf formula:
```
slot_width = material_thickness - kerf
finger_width = (panel_length / num_fingers) - kerf
```

---

### CorelDRAW (.cdr, SVG, DXF, AI, PDF)

Colour convention:

| Colour | Operation |
|--------|-----------|
| Red RGB #FF0000 | Vector CUT (hairline stroke) |
| Blue RGB #0000FF | Vector ENGRAVE line (hairline stroke) |
| Black RGB #000000 | Fill ENGRAVE (raster) |
| Greyscale fills | Raster engrave depth: black=deepest, white=nothing |

CorelDRAW specific rules:
- Stroke width for cut/score: Hairline or 0.01mm
- Cut objects: NO fill, stroke = hairline red
- Engrave objects: fill = black (or greyscale), NO stroke
- Text: MUST convert to curves before export (Ctrl+Q in CorelDRAW)
- Stroke position: Centre (not inside/outside) — critical for dimensional accuracy
- Remove all effects: shadows, lens, contour, extrude — not supported by laser software
- Bitmap trace: use Outline Trace > High Quality Image for photos
- After trace: delete original bitmap, keep only vector outline

File prep steps:
1. Set document size = material size (mm)
2. Cut objects: no fill, hairline red stroke
3. Engrave objects: black fill, no stroke
4. Text: Ctrl+Q convert to curves
5. Check for duplicate lines
6. Print/Send to laser printer driver

---

## 2. KERF COMPENSATION MATHS

Kerf = width of material removed by the laser beam.

```
Typical kerf values:
- Diode laser 5-20W:   0.10–0.20mm
- CO2 40W:             0.15–0.25mm
- CO2 60-80W:          0.20–0.30mm
- CO2 100W+:           0.25–0.40mm
- Fibre laser:         0.05–0.15mm
```

Machine kerf library for Artfull:
```typescript
export const MACHINE_KERF_MM = {
  'xtool-d1-pro-20w':  0.12,
  'xtool-d1-pro-10w':  0.15,
  'sculpfun-s30-pro':  0.12,
  'sculpfun-s9':       0.15,
  'creality-falcon2':  0.13,
  'atomstack-a20-pro': 0.14,
  'atomstack-x20-pro': 0.13,
  'k40-co2':           0.20,
  'ortur-lm3':         0.14,
  'twoTrees-ts2':      0.13,
  'custom':            null,
}
```

Kerf compensation formula:
```typescript
export function applyKerf(dimension: number, kerf: number, side: 'external' | 'internal'): number {
  if (side === 'external') return dimension - kerf
  else return dimension + kerf
}

export function fingerJointWidth(panelLength: number, numFingers: number, kerf: number) {
  const rawWidth = panelLength / numFingers
  return {
    finger: rawWidth - kerf,  // finger slightly narrower
    slot:   rawWidth + kerf,  // slot slightly wider
  }
}
```

---

## 3. FINGER JOINT BOX — ALGORITHM

```typescript
interface BoxParams {
  width: number      // internal X dimension (mm)
  height: number     // internal Z dimension (mm)
  depth: number      // internal Y dimension (mm)
  thickness: number  // material thickness (mm)
  kerf: number       // laser kerf (mm)
  numFingers?: number
}

// Auto-calculate optimal fingers: width of each finger ~ 3x material thickness
// Always use odd number for symmetry
const fingers = (() => {
  const auto = Math.max(3, Math.round(params.width / (params.thickness * 3)))
  return auto % 2 === 0 ? auto + 1 : auto
})()

// Panel dimensions:
// Bottom/top:    width × depth
// Front/back:    width × height
// Left/right:    (depth - 2×thickness) × height  ← CRITICAL: inset side panels

// Minimum sheet size:
const sheetWidth  = width + depth + thickness * 4 + 5   // 5mm margin
const sheetHeight = (height + depth) * 2 + thickness * 4 + 10
```

---

## 4. MAKER.JS INTEGRATION

```typescript
import makerjs from 'makerjs'

export function modelToSVG(model: makerjs.IModel): string {
  return makerjs.exporter.toSVG(model, {
    units: makerjs.unitType.Millimeter,
    layerOptions: {
      'cut':     { stroke: '#FF0000', strokeWidth: '0.1' },
      'engrave': { stroke: '#0000FF', strokeWidth: '0.1' },
    },
    strokeOnly: true,
  })
}

export function modelToDXF(model: makerjs.IModel): string {
  return makerjs.exporter.toDXF(model, {
    units: makerjs.unitType.Millimeter,
    pointMatchingDistance: 0.005,
  })
}
```

---

## 5. COMMON ERRORS — FIX GUIDE

| Problem | Cause | Fix |
|---------|-------|-----|
| Parts don't fit | Kerf not compensated | Apply applyKerf() to joint dimensions |
| Laser cuts twice | Overlapping paths | Use makerjs.model.combineUnion() |
| Wrong size in LightBurn | SVG missing mm units | Set width="200mm" not width="200" |
| DXF imports as bitmap | File is raster | Only DXF vector for cutting |
| Spline error | Fusion exported splines | Convert to arcs before export |
| Text wrong | Live text in file | Always convert to paths/curves |
| Box corners misalign | Wrong panel formula | Side panels must be depth - 2×thickness |
| Scale 10x wrong | DXF in wrong units | Set DXF units to mm explicitly |

---

## 6. VALIDATION — ALWAYS RUN BEFORE GENERATING

```typescript
export function validateLaserParams(params: {
  width: number; height: number; depth?: number
  thickness: number; kerf: number
}): string[] {
  const errors: string[] = []
  if (params.width <= 0 || params.height <= 0)
    errors.push('Dimensões devem ser positivas')
  if (params.thickness < 1)
    errors.push('Espessura mínima: 1mm')
  if (params.thickness > 25)
    errors.push('Espessura máxima suportada: 25mm')
  if (params.kerf < 0)
    errors.push('Kerf não pode ser negativo')
  if (params.kerf > 2)
    errors.push('Kerf superior a 2mm — verifica o valor')
  if (params.depth && params.depth <= params.thickness * 2)
    errors.push('Profundidade deve ser superior a 2× espessura do material')
  if (params.width < params.thickness * 6)
    errors.push(`Largura mínima para finger joints: ${params.thickness * 6}mm`)
  return errors
}
```

---

## 7. MATERIAL REFERENCE

| Material | Thickness | Kerf CO2 | Min feature | Notas |
|----------|-----------|----------|-------------|-------|
| MDF | 3, 6mm | 0.20mm | 3mm | Absorbe humidade |
| Contraplacado bétula | 3, 4, 6mm | 0.18mm | 4mm | Melhor para joints |
| Acrílico | 3, 5mm | 0.15mm | 5mm | Cast > extruded |
| Cartão 3mm | 3mm | 0.10mm | 2mm | Baixa potência |
| Couro | 2–4mm | 0.12mm | 3mm | Não usar couro cromado |
| Feltro | 2–3mm | 0.08mm | 2mm | Baixa potência, rápido |

NUNCA cortar com laser:
- PVC (gás cloro — extremamente tóxico)
- Policarbonato / PC (fumos tóxicos)
- HDPE (derrete, não corta)
- Qualquer material com cloro ou fluoretos

---

## 8. SVG OUTPUT STANDARD — ALWAYS FOLLOW

Every SVG output must:
- viewBox in mm: viewBox="0 0 {width} {height}"
- Width/height with mm unit: width="{width}mm"
- Cut paths: stroke="#FF0000", fill="none", stroke-width="0.1"
- Engrave paths: stroke="#0000FF", fill="none", stroke-width="0.1"
- No embedded fonts — all text converted to paths
- No effects, gradients, or opacity
- Clean path data

Every DXF output must:
- Version: R14 (AC1014)
- Units: METRIC (mm)
- Layers: "CUT" (colour 1), "ENGRAVE" (colour 5)
- Entities: LWPOLYLINE for closed shapes
- All Z coordinates = 0
- Header: INSUNITS=4 (millimetres)

---

## 9. PHOTO FRAME — PROVEN DESIGN (4 pieces)

Validated design for laser-cut photo frames:

```
SANDWICH (side view):  FRENTE | ESPAÇADOR | TRASEIRA
Photo slides in from top through the spacer channel.
```

### Pieces:
1. **FRENTE** — outer rect + window (photo - 3mm overlap each side). Overlap holds the photo.
2. **ESPAÇADOR** — outer rect + window (photo + 1mm clearance), OPEN at top. Creates the channel depth for the photo to slide into.
3. **TRASEIRA** — solid panel + vertical slot (centred, from bottom, 35% height) + 2 locking slots (wider than main slot, positioned at 35% and 65% along the slot).
4. **SUPORTE** — kickstand shape:
   - Wider at bottom (40% of frame width), tapers to top (28%)
   - Tab on top (width = material thickness, length = 85% of slot)
   - Rectangular locking notch on left side (catches in the locking slots on the back)
   - Body tapers linearly; notch is a rectangular step cut into the left edge

### Assembly:
- Glue Frente + Espaçador + Traseira
- Photo slides in from top
- Support tab goes into back slot, locking notch clicks into position

### Key dimensions:
- Window overlap: 3mm (photo sits on ledge, doesn't fall)
- Spacer clearance: 1mm (photo slides freely)
- Slot width: material thickness
- Locking slots: slot width + 2× material thickness (wider)
- Support tab: same width as slot, 85% of slot depth

### Preview rules:
- Show photo zone (light blue fill) ONLY in preview, NOT in download SVG
- Labels in black, font-size 5-6mm in preview for readability
