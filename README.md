# THREADLIGHT v2
### Elastic Glow × Hand Games

MediaPipe hand tracking + elastic spring web + 4 interactive modes.

---

## Modes

| Mode | Gesture | Action |
|---|---|---|
| **ELASTIC GLOW** | Any | Pure visual — spring web, neon bloom |
| **DUCK HUNT** | Pinch / Point finger | Shoot flying ducks 🦆 |
| **ORB BLAST** | Pinch = shoot · Open palm = blast | Destroy falling orbs 🔮 |
| **LIGHT PAINTER** | Open = paint · Fist = clear | Draw with glowing trails 🎨 |

---

## Gesture Reference

| Gesture | Detection | Used in |
|---|---|---|
| **PINCH** | Thumb + index tips < 6% apart | Duck Hunt, Orb Blast — fires bullet |
| **POINT** | Only index finger extended | Duck Hunt, Orb Blast — fires bullet |
| **OPEN** | 4 fingers extended | Orb Blast — palm blast radius |
| **FIST** | No fingers extended | Painter — clears canvas |

---

## Run Locally

```bash
# Python (no install needed)
cd threadlight-v2
python3 -m http.server 8080
# Open http://localhost:8080 in Chrome
```

```bash
# Node.js
npx serve .
```

> Must run over HTTP, not `file://` — camera API requires a server origin.
> Chrome or Edge recommended for best MediaPipe performance.

---

## Deploy to Vercel

### CLI (fastest)
```bash
npm install -g vercel
cd threadlight-v2
vercel --prod
```
When prompted:
- Framework preset: **Other**
- Build command: *(leave empty)*
- Output directory: `.` (dot)

### GitHub → Vercel (easiest for sharing)
1. Create repo and push:
```bash
git init
git add .
git commit -m "THREADLIGHT v2"
git remote add origin https://github.com/YOUR_USER/threadlight.git
git push -u origin main
```
2. Go to [vercel.com](https://vercel.com) → **Add New Project** → import repo
3. Framework: **Other** → Deploy

`vercel.json` handles:
- `Permissions-Policy: camera=*` — allows camera on the deployed URL
- COEP/COOP headers — required for SharedArrayBuffer

---

## Files

```
threadlight-v2/
├── index.html    ← entire app (zero dependencies, self-contained)
├── vercel.json   ← Vercel deployment config
└── README.md     ← this file
```

---

## How it works

### TD Signal Chain → Web
| TouchDesigner | Web |
|---|---|
| MediaPipe Base COMP + In CHOP | `@mediapipe/hands` via CDN |
| Math 1 Post-Add −0.5 | `tx = lm.x - 0.5` |
| Math 2 ty × −0.5625 | Flip Y + 16:9 aspect correction |
| Noise CHOP r=0.9 g=0.3 b=0.6 | `sin()`-based noise per landmark |
| Proximity POP r=0.15 max=42 | Screen-space distance check |
| Spring POP K=65 D=15 | Euler spring integration |
| Blur 6 + Blur 20 | CSS `filter:blur()` on offscreen canvas |
| Composite Add ×2 | `globalCompositeOperation='screen'` × 3 |
| Color Correct Hue 340 Sat 1.4 | RGB saturation multiply |
| Video In → Screen blend | Camera feed at low alpha |

### Game additions
- **Gesture detection** — per-hand landmark topology analysis (no ML, pure geometry)
- **Spring bullets** — fired from index tip toward wrist→tip direction vector
- **Palm blast** — landmark 9 (palm center) radius check against all targets
- **Combo system** — consecutive hits multiply score, decay on miss
- **Particle FX** — burst spawner on hit, additive blended
- **Trail canvas** — separate canvas with slow fade for motion trails
