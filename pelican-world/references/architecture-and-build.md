# Architecture and build

## Why one HTML file with three.js inlined

- **Offline play:** In mainland China, without a VPN, CDNs (unpkg, jsDelivr, Google Fonts) may not load. three.js must be bundled; fonts are a system stack; images are painted in code or inlined as `data:` URLs.
- **Simple distribution:** One file, double-click to play, or drop onto any static host.
- **Hard test:** Launch the browser context with `offline: true` and record every request that is not `file:`, `data:`, or `blob:`. That list must be empty.

## Suggested tree

The author's documented layout (adapt names to the new game, keep the split):

```
project/
├─ template.html        # HTML + CSS (UI, HUD, screens), with two placeholders
├─ build.mjs            # esbuild bundles src/main.js and inlines it into the template
├─ thumbs/*.jpg         # optional stage-select thumbnails, inlined as base64 at build time
├─ src/
│  ├─ main.js           # renderer, state machine, input, camera, HUD, quality, loop
│  ├─ world.js          # terrain, road, water, particles, obstacles, kit()
│  ├─ regions.js        # per-stage data + build() (sky, grade, terrain, landmarks)
│  ├─ rider.js          # hero / vehicle modeling and procedural animation
│  ├─ props.js          # reusable props (trees, lamps, benches, landmarks)
│  ├─ audio.js          # Web Audio procedural music + SFX
│  ├─ textures.js       # Canvas textures (roads, facades, signs, sprites)
│  ├─ map.js            # optional 2D route map
│  ├─ util.js           # RNG, noise, fbm, merge geometry, vertex AO
│  ├─ gfx/              # sky, post, water, surfaces, globals, quality
│  └─ detail/           # high-detail painters and per-city generators
└─ *.mjs                # Playwright tests: shots, logic, audio, UI, …
```

Documented line counts for the author's v3 tree (notes only, not re-counted here) are in the file map at the end of this page. Total documented game source is about 3,250 lines.

## Build pipeline

The documented `build.mjs` is short. The important behaviors are:

```js
import * as esbuild from 'esbuild';
import fs from 'fs';

const r = await esbuild.build({
  entryPoints: ['src/main.js'],
  bundle: true,
  minify: true,
  format: 'iife',
  write: false,
  target: 'es2020',
  legalComments: 'inline',
  banner: {
    js: '/*! Your Game Title — © year author. License notices go here. */'
  }
});

let js = r.outputFiles[0].text.replace(/<\/script/gi, '<\\/script');
const tpl = fs.readFileSync('template.html', 'utf8');
const out = tpl.replace(
  '<script>/*__GAME__*/</script>',
  () => '<script>\n/* three.js (MIT License, (c) 2010-2026 three.js authors) bundled inline */\n' + js + '\n</script>'
);

let thumbs = {};
try {
  for (const f of fs.readdirSync('thumbs')) if (f.endsWith('.jpg')) {
    thumbs[f.slice(0, -4)] = 'data:image/jpeg;base64,' + fs.readFileSync('thumbs/' + f).toString('base64');
  }
} catch (e) { /* thumbs directory may be absent early on */ }

const out2 = out.replace(
  '<script>/*__THUMBS__*/</script>',
  () => '<script>window.__THUMBS=' + JSON.stringify(thumbs) + '</script>'
);

fs.writeFileSync('game.html', out2);
console.log('thumbs', Object.keys(thumbs).length);
console.log('bytes', Buffer.byteLength(out2));
```

Practices that matter:

- `format: 'iife'` so the bundle does not depend on ES module loading in a `file://` page.
- Pass a **function** as the `String.prototype.replace` replacement so `$&` or `$1` inside the JavaScript is not treated as a replacement pattern.
- Escape `</script` in the bundle or a string inside three.js or a shader can close the tag early.
- `legalComments: 'inline'` plus `/*! ... */` keeps copyright and the three.js MIT notice in the minified output.
- Log `Buffer.byteLength` of the **final written file**, not an intermediate buffer. The notes record a pitfall where the logged size was not the artifact on disk.
- Rebuild after every increment so the HTML on disk is always playable.

Import only the three.js plugins you use (the documented game used `SunLight` and `BufferGeometryUtils`).

## Test hooks and the review loop

Expose a small `window.__game` API so tests can drive the game: start a stage, read frame count, force a camera cut, set quality, load a region. The documented shape included helpers such as `startStage`, `frames()`, `cut()`, `setQuality`, `loadRegion`, and handles to `world`, `audio`, and `REGIONS`.

The review loop is:

1. `node build.mjs` and read the printed byte length.
2. Screenshot each region from fixed cameras (chase, street, water, landmark).
3. Shrink shots (the notes used 640×360) into a contact sheet so one region can be judged at a glance.
4. Write down what looks cheap (flat emissive windows, milky fog, tiling, overexposed shop windows, spherical trees).
5. Change one or two parameters or one generator, then go back to step 1.

Details of Playwright flags, waiting on frame count, and the auto-play script are in [Testing and iteration](testing-and-iteration.md).

## Quality tiers and adaptive resolution

The documented game used four tiers. Each tier jointly controls resolution scale, shadow map, AO, god rays, reflection, motion blur, and particle count:

```js
export const TIERS = [
  { name: '流畅', scale: .72, maxDpr: 1,    budget: 1.0e6, msaa: 0, shadow: 1024, shadowFar: 110, shadowEvery: 2, ao: false, aoTaps: 8,  rays: false, raySteps: 24, refl: 0, reflScale: 0,   motion: false, bloom: true, particles: .45 },
  { name: '均衡', scale: .85, maxDpr: 1.25, budget: 1.6e6, msaa: 0, shadow: 2048, shadowFar: 150, shadowEvery: 1, ao: true,  aoTaps: 8,  rays: true,  raySteps: 28, refl: 2, reflScale: .35, motion: false, bloom: true, particles: .75 },
  { name: '精美', scale: 1,   maxDpr: 1.5,  budget: 2.4e6, msaa: 4, shadow: 2048, shadowFar: 190, shadowEvery: 1, ao: true,  aoTaps: 12, rays: true,  raySteps: 40, refl: 1, reflScale: .5,  motion: true,  bloom: true, particles: 1 },
  { name: '极致', scale: 1,   maxDpr: 2,    budget: 3.7e6, msaa: 4, shadow: 4096, shadowFar: 230, shadowEvery: 1, ao: true,  aoTaps: 16, rays: true,  raySteps: 56, refl: 1, reflScale: .75, motion: true,  bloom: true, particles: 1 },
];
```

Documented selection and adaptation:

- **Initial tier** from `WEBGL_debug_renderer_info`. Names matching `swiftshader|llvmpipe|software` or low-end `mali/adreno|powervr` start at tier 0; `intel|mali|adreno|apple gpu|iris|uhd` start at 1; anything else starts at 2. Touch devices cap at 1. A URL query `?q=0..3` forces a tier.
- **Pixel budget:** if `W×H×scale²` exceeds `budget`, shrink the render scale by `sqrt(budget/px)` and run post on the smaller RT.
- **Dynamic resolution:** wait 3 seconds to settle, then use a 40-frame or 1-second average frame time. Above 26 ms, drop scale by 0.1 (by 0.2 if above 40 ms); below 17.5 ms, raise by 0.1; floor at 0.6. If the scale is already 0.6 and the average stays above 34 ms twice in a row, drop a quality tier.
- **Fallback:** if WebGL2 or float color buffers are missing (`canPost = false`), disable the custom post stack and use three.js `ACESFilmicToneMapping` so the game still runs.
- Shadow maps update every `shadowEvery` frames (`renderer.shadowMap.autoUpdate = false`, set `needsUpdate` yourself).

## Documented file map (Pelican World v3 notes)

Line counts are from the author's notes, not re-counted in this package.

| File | Lines (documented) | Role |
|---|---|---|
| `src/main.js` | 333 | Renderer, tiers, post hook-up, FX, state, input, camera, HUD, loop, test hooks |
| `src/world.js` | 357 | Path, terrain, road, water, clouds, particles, `kit()`, gameplay layout, obstacles |
| `src/regions.js` | 302 | Six stages: data + `build()` |
| `src/rider.js` | 332 | Bicycle and pelican modeling and animation |
| `src/props.js` | 207 | Trees, lights, landmarks, reusable props |
| `src/textures.js` | 131 | Road and sprite canvases |
| `src/audio.js` | 85 | Procedural music and SFX |
| `src/map.js` | 49 | Route map |
| `src/util.js` | 80 | RNG, noise, merge, AO bake, geometry helpers |
| `src/gfx/sky.js` | 201 | Physical sky, clouds, stars, moon, IBL, JS atmosphere |
| `src/gfx/post.js` | 398 | Entire post stack |
| `src/gfx/water.js` | 164 | Planar reflection, water, wet ground |
| `src/gfx/surfaces.js` | 115 | Procedural normals/roughness, triplanar detail |
| `src/gfx/globals.js` | 80 | Fog shader chunks, PCF rewrite |
| `src/gfx/quality.js` | 16 | Tiers and GPU detection |
| `src/detail/paint.js` | 52 | Three-layer canvas helpers |
| `src/detail/paris.js` | 194 | Haussmann block generator |
| `src/detail/nyc.js` | 154 | New York block and street generator |
| `template.html` | ≈34 KB | All UI |
| `build.mjs` | — | Production HTML build |

## Key numeric defaults (documented)

A fuller table lives in [Checklists](checklists.md). Highlights:

| Area | Values |
|---|---|
| Camera near / far / FOV | 0.15 / 3200 / `58 + speed * 0.55` |
| Follow distance / height | `5.6 + speed * 0.05` / `2.5 + y * 0.4` |
| Damping λ (steer / lean / cam pos / cam look / FOV) | 9 / 8 / 7 / 12 / 3 |
| Dynamic resolution down / up / floor | >26 ms / <17.5 ms / 0.6 |
