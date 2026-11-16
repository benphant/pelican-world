# Materials

The documented recipe for surface quality: **paint in three layers** (base shape, noise and grime, light and edges) and **use more noise**. Roughness variation sells material more than color variation.

## Noise toolbox (documented as authored in-repo, no noise library)

| Function | Where | Use |
|---|---|---|
| `makeRng(seed)` | `util.js` | mulberry32-style deterministic RNG. **The same seed must rebuild the same world**, or screenshot comparisons are meaningless |
| `noise2` / `fbm(x, y, oct)` | `util.js` | Value noise with a permutation table and smoothstep interpolation; fbm frequency ×2.03 per octave (not ×2, so octaves do not align to a grid), amplitude ×0.5 |
| `vnoiseT` / `fbmT(x, y, P, oct)` | `gfx/surfaces.js` | Value noise with period `P` (coordinates mod P), for seamless textures |
| `fbmWrap` + `noiseT` | `world.js` | Ground normals: 5 tileable fbm octaves, each multiplied by a ridge term `1 - .35*\|fbm*2-1\|` |
| `noise()` | `detail/paint.js` | Multi-octave noise on a Canvas: upsample a small random image with high-quality bilinear, layer with n doubled and alpha ×0.62; supports `overlay` / `multiply` / `soft-light` |
| `hash12` / `ign` | `gfx/post.js` | White noise and interleaved gradient noise in shaders (AO rotation, grain, dither) |
| Shader `fbm` | `gfx/sky.js`, `gfx/post.js` | 5 octaves, `p = p*2.03 + 11.3` per octave, for clouds and lens dirt |

**Optional (not documented as used in Pelican World v3):** Perlin or Simplex noise, or Worley (cellular) noise for more organic rock or cloud. The shipped example used value noise plus fbm.

## Three-layer canvas paint

`detail/paint.js` `layeredSet(w, h, paint)` returns **four aligned canvases**: albedo `a`, height `h` (default `#808080`), roughness `r` (default `#d0d0d0`), emissive `e` (default black). One paint function draws all four, so brick joints line up in color, bump, and roughness. Convert the height map with `heightNormal(canvas, 5)`.

**Layer 1 — base shapes.** Flat color plus geometry: bricks, window holes, doors, sign boards.

**Layer 2 — noise and grime,** to break the "drawn in software" look:

```js
noise(a, 512, 512, { cells: 4, oct: 5, alpha: .38, seed: 12 });                  // broad color variation (overlay)
noise(a, 512, 512, { cells: 2, oct: 3, alpha: .3, seed: 14, mode: 'multiply' }); // larger dirt
speckle(a, 512, 512, 3000, ['rgba(255,240,220,.14)', 'rgba(0,0,0,.2)'], 6, .6, 1.8);
streaks(a, 512, 512, 300, 33, 'rgba(10,8,6,.12)', [20, 160]);                     // 300 vertical rain streaks
```

**Layer 3 — light and edges,** so a flat quad reads as a volume:

```js
// Raised panel: bright line on top/left, dark line on bottom/right, as if lit from above-left
export function bevel(g, x, y, w, h, fill, k = 1,
  hi = 'rgba(255,255,255,.28)', lo = 'rgba(0,0,0,.35)') { /* … */ }
// Gradient shadow under a sill:
const sg = a.createLinearGradient(0, wy + 97, 0, wy + 128);
sg.addColorStop(0, 'rgba(0,0,0,.28)');
sg.addColorStop(1, 'rgba(0,0,0,0)');
```

### Example: New York brick (512² = 4 bays × 4 floors)

1. Base `#5e3328`. Draw each brick (6 px high, 16 px wide, running bond) with a slight random color jitter. On the height map, brick faces are `#9a9a9a`; mortar stays the default grey and sits lower.
2. Two `noise` passes, then `speckle`.
3. Each window cell:
   - Beveled frame; glass is a top-to-bottom dark gradient.
   - **About 24% of windows lit**, picked from five warm/cool whites. The emissive layer is painted only in lit panes, with a vertical gradient (brighter at the top).
   - Random blinds: horizontal lines that also mask emissive.
   - 16% of windows get a beveled AC box with a grille.
   - Pane roughness is very dark `#141414`, so glass reflects.
4. Stone lintels and sills use `bevel`, a contact shadow under the sill, then 300 rain streaks.

The same three-layer idea is applied to UI glass in [UI](ui.md).

## Documented PBR starting points

These are values from the example game, not universal constants.

| Material | Key parameters |
|---|---|
| Pelican white feathers | `MeshPhysicalMaterial`, `roughness .78`, procedural feather normal/roughness, `sheen .85`, `sheenColor #fffaf0` (sheen is what reads as down or cloth) |
| Beak | `roughness .32, clearcoat .5` — oily keratin |
| Eye | `clearcoat 1, clearcoatRoughness .02` — one highlight that sells life |
| Helmet | `roughness .28, clearcoat 1, clearcoatRoughness .05` — paint |
| Scarf | `roughness .9`, woven normal (`fabricTextures`), `sheen 1` |
| Road | `roughness .85`, normal from road-canvas luminance (`canvasNormal`; cobble/flagstone strength 6, asphalt 3.5) |
| Terrain | vertex color + detail map + ground normal `normalScale .45`, `roughness .93` |
| Paris signs | `metalness .85, roughness .25, alphaTest .45` — gilt letters |
| Tower lattice | `#7a6048, metalness .45`, `alphaTest` cutouts |
| Water | `MeshPhysicalMaterial`, `ior 1.333, roughness .05`, premultiplied alpha |
| Neon | additive, color ×2.6, `toneMapped: false` |

Experience from the notes:

- Cloth and feathers use `sheen`, not metalness.
- Paint, eyes, and wet surfaces use `clearcoat`.
- **Roughness variation does more than color variation.** Give every surface a roughness map, or perturb roughness in the triplanar detail.

## Procedural normals and roughness

- `heightToNormal(h, N, strength)`: any height field to a tileable normal map (central differences, periodic boundaries).
- `canvasNormal(canvas, strength)`: treat a painted color map as height (bright = raised), 3×3 blur, then a normal. Every hand-painted road or facade gets bump "for free."
- `featherTextures()`: a per-vane distance field (shaft + barbs + tip fade) → normal and roughness.
- `fabricTextures()`: warp/weft `sin` displacement.
- `detailNormalTex()`: general micro-detail, two `fbmT` layers (periods 8 and 32); RG stores slope, B stores height / cavities.

## World-space triplanar micro-detail (merged geometry has no reliable UVs)

Props are many primitives merged and colored with vertex colors (see [World building](world-building.md)). There is no trustworthy UV. Patch the shared material `world.vcMat` in `onBeforeCompile` and sample a micro-detail map in **world space** with triplanar projection:

```js
world.vcMat = triplanarDetail(
  new MeshStandardMaterial({ vertexColors: true, roughness: .78 }),
  detailN, 1.3, .32, .3, .2
);
// texture, scale, normal strength, roughness jitter, cavity darkening
```

The shader then stacks four "weathering" terms (this is the source of the worn look):

```glsl
diffuseColor.rgb *= 1.0 - uTpCav * (1.0 - smoothstep(0.2, 0.7, tpH));           // cavities darker
float big = texture2D(uTpTex, vTpW.xz * 0.013 + vTpW.y * 0.004).z * 0.6 + /* … */;
diffuseColor.rgb *= 0.86 + 0.28 * big;                                          // 1. large albedo drift
float wall = 1.0 - abs(vTpN.y);
float streak = texture2D(uTpTex, vec2(dot(vTpW.xz, vec2(.71)) * 0.9, vTpW.y * 0.035)).z;
diffuseColor.rgb *= 1.0 - wall * uTpGrime * smoothstep(0.45, 0.8, streak) * 0.35; // 2. vertical rain, Y×25
float gnd = 1.0 - smoothstep(0.0, 0.9, vTpW.y - uTpGround);                       // 3. 0.9 m splash zone
diffuseColor.rgb = mix(diffuseColor.rgb, diffuseColor.rgb * vec3(0.86, 0.8, 0.72), gnd * wall * uTpGrime * 0.6);
roughnessFactor = clamp(roughnessFactor * (1.0 + (tpH - 0.5) * 2.0 * uTpRough), 0.04, 1.0); // 4. roughness
normal = normalize(normal + mat3(viewMatrix) * tpP * uTpStr);
```

- Blend weights `pow(abs(N), 4)` then normalize, for sharper seams.
- Sample two scales (second ×0.23, offset 0.37) to break tiling.
- Support `InstancedMesh`: multiply `instanceMatrix` in the vertex shader first.
- Set `customProgramCacheKey`, or three.js will reuse one program for patched materials that actually differ.

## Vertex-color AO bake plus albedo noise

When `mergeColored(parts)` merges a prop, each vertex gets a cheap "fake AO" by default:

```js
let ao = .6 + .4 * smoothstep(0, .55, t);
// t = relative height inside the object: darker at the base (grounding)
if (ny < -.3) ao *= .72; else if (ny > .6) ao *= 1.04; // down-faces darker, up-faces lighter
const v = .9 + .2 * fbm(x*.35 + 17.3, z*.35 + y*.2, 3)
        + (fbm(x*2.1, z*2.1 + y*1.7, 2) - .5) * .08;   // world-space color noise
```

Nearly free, and every low-poly prop then has contact darkening and is not a pure color. It stacks with screen-space SSAO: one for large scale, one for creases.

## Instancing for density

- `kit().instanced(geom, mat, list)`: each list entry is `[x, y, z, ry, scale, color]`, one draw for hundreds or thousands of objects. Examples: café tables, 260 snow-capped boulders, hydrants, 64 chain links, cloud puffs.
- Color variation via `setColorAt`, so one mesh has many colors.
- Rule from the notes: **instance anything that repeats ~20+ times**; only unique landmarks stay as standalone meshes.

## Texture checklist (run on every screenshot)

- [ ] Any large flat color? Add layer-2 noise.
- [ ] Are wall bases, eaves, and sills darker? Add layer-3 contact shadow or AO.
- [ ] Is the highlight identical everywhere? Vary roughness.
- [ ] Obvious tiling or a grid? Use dual-scale sampling, or remove regular `sin` terms (a documented ground-normal bug was a `sin(u)·sin(v)` term that showed as a grid at grazing angles).
- [ ] Are emitters "blown-out rectangles"? Multiply emissive by albedo and lower the intensity.
- [ ] Does a close-up look too smooth? Add micro-normals, triplanar detail, or `canvasNormal`.
