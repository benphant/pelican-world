# World building

## Data-driven regions

Each stage is one object: pure data on top, a `build(K, rng)` function below. A new stage is a new object; engine code does not change.

```js
{ id: 'nyc', name: '纽约', en: 'NEW YORK · USA', landmark: 'Statue of Liberty',
  lat: 40.69, lon: -74.04, time: '璀璨夜晚',
  fact: '…', length: 1350,                         // stage length in meters
  path: { a1: 14, f1: .004, p1: 1.4, a2: 6, f2: .012, p2: 0 }, // road bend = two sines
  road: 'night', curb: '#9aa0a8', wet: { puddles: .48 }, reflY: .04,
  water: { x: 0, y: -.25, z: -1950, w: 1400, l: 900, deep: '#06101c', shallow: '#13263a' /* … */ },
  particles: null,
  obstacles: ['taxi', 'hydrant', 'trash', 'cone', 'barrier', 'taxi'], density: 1.2,
  terrain: (x, z, d, sd) => { /* height; d = distance to road centerline */ },
  groundColor(c, x, z, h, d) { /* vertex colors */ },
  sky: { /* sunDir, ray, mie, fog, moonLight, fill, envI, night, … */ },
  grade: { /* shadowTint, highTint, sat, contrast, key, bloom, dirt, rays */ },
  music: { style: 'jazz', bpm: 132, root: 58, scale: [/* … */], prog: [[/* … */]], lead: 'piano' },
  build(K, rng) { buildNYBlocks(K, /* … */); nycStreet(K, /* … */); K.mesh(liberty(), /* … */); }
}
```

Stage order is one line. The documented example is:

```js
export const REGIONS = ['nyc', 'china', 'japan', 'hawaii', 'egypt', 'paris'].map(/* … */);
```

The author put the strongest stage first and a then-weak stage last by editing only that line. The method is: **finish one flagship stage to a high standard, then copy the pattern.**

Landmark-like props are in-game reconstructions, not official or licensed depictions. Do not describe them that way to the user or in credits.

## Road coordinates (s, u)

Place everything in "distance along the road `s`, lateral offset `u`", then convert to world space:

```js
cx(s) {
  const ramp = smoothstep(0, 140, s);
  return ramp * (a1*sin(s*f1+p1) + a2*sin(s*f2+p2) - startOffset);
} // first 140 m is straight
point(s, u) {
  const a = dcx(s), n = Math.hypot(1, a);
  return V(cx(s) + u/n, 0, -s + u*a/n); // offset u along the normal
}
yaw(s) { return -Math.atan(dcx(s)); }
```

- Player, obstacles, collectibles, ramps, and roadside buildings all use `(s, u)`, so buildings automatically follow a bend.
- Documented road half-width `ROAD_HW = 5`, three lanes `u ∈ {-3, 0, 3}`.
- Roadside facades use `frame(K, s, uFront, side)` to build a right-handed basis along the road (`makeBasis(up × n, up, n)`), so walls face the road.

## Terrain, road, water

- **Terrain:** a 900 m-wide `PlaneGeometry`, subdivided every 5 m (8 m on the low tier). Height from `region.terrain(x, z, |d|, d)`, color from `groundColor` (vertex colors). Overlay a detail map and a tileable ground normal, `normalScale .45`.
- **Wilder farther from the road.** Great Wall example:

```js
h += smoothstep(12, 40, d) * (
  (fbm(x*.07, z*.07) - .5) * 4.5
  + Math.pow(Math.abs(fbm(x*.03, z*.03) * 2 - 1), 2) * 5
); // flat near the road, rolling in the distance, with ridges
```

- **Road:** a cross-section every 2 m, width ±5 m, UVs tile a Canvas road texture along `s`. Shoulders are a 4-vertex curb profile per side (vertex colors).
- **Water, two kinds:**
  - A strip that follows the road (`band: [u0, u1]`).
  - A standalone rectangle (sea, lake, moat).
  - Both write `aDepth = waterHeight - terrainHeight` for color absorption and shore foam.
- Hand-built strips can end up with reversed winding. After `computeVertexNormals()`, check that `normal.getY(0)` is positive; the documented river strip corrects itself this way. When a generator's parameter order flips facing, confirm **both** sides in screenshots.

## Props: primitives + vertex color + merge

Trees, lamps, landmarks, and obstacles are assembled from three.js primitives (Box, Cylinder, Sphere, Cone, Torus, Lathe, Extrude). Each piece gets a color and a transform, then `mergeColored()` merges them:

```js
const p = [];
p.push([new THREE.CylinderGeometry(.16, .16, .6, 12), '#d62828', mat4(0, .3, 0)]);
// …
return mergeColored(p); // merge + vertex AO + noise → 1 draw, shared world.vcMat
```

- **One material for almost every prop:** vertex colors plus triplanar detail. Draw calls stay low.
- **Geometry cache:** `world.geo(name, fn)` builds a named geometry once and reuses it (needed for instancing).
- `tubeBetween(a, b, r0, r1)`: a tapered tube between two points (legs, rails, branches, trusses).
- `jitterGeom(g, amt, seed)`: vertex jitter so rocks and clouds are not perfect.

## High-detail generators (v3 Paris and New York)

The v3 approach is **to model a facade in architectural layers**, not a textured box.

**Paris Haussmann blocks** (`buildParisBlocks`):

- **Facade atlas:** 512² = 4 bays × 2 floors. Ground floor 4.4 m, typical floor 3.15 m, bay about 3 m. Each building gets its own UVs so window proportions stay correct.
- **Shopfronts:** a 1024×256 atlas, 4 variants, with emissive interiors; gilt serif signs (metalness 0.85); striped awnings (8 stripes, pitched 0.3–0.38 rad, 1.7–2.6 m deep) plus a valance.
- **Balconies:** continuous on the second and top floors: stone slab, corbels, `alphaTest` iron rail (`DoubleSide`), flower boxes.
- **Moldings and roofs:** string courses per floor, dentiled cornice; **zinc mansard** as two truncated pyramids (inset the top vertices of a box), with dormers (triangular prism caps) and chimneys (clay pots).
- **Street furniture:** instanced café sets, Morris columns, bollards, plane trees (26 jittered leaf clumps on an ellipsoid, 5 primary limbs, mottled bark).
- **Tower:** each face of each major beam carries an `alphaTest` X-lattice.

**New York pre-war blocks** (`buildNYBlocks`):

- Facades: brick or limestone (see [Materials](materials.md)); ground-floor deli / bar / pizza / graffitied shutter atlas.
- Signs: additive **neon** atlas (Canvas `shadowBlur` strokes); blade signs.
- Façade kit: zigzag **fire escapes** (landings, rails, stairs); rooftop **wooden water towers**; set-back towers above 52 m.
- Street (`nycStreet`): cycling traffic lights; Con Edison red-and-white steam pipes with animated steam sprites; hydrants, call boxes, trash cans; a sidewalk light pool in front of each shop window.

**Density rule from the notes:** in the 5–30 m band around the camera, there should be something readable at least every 3 m (a window, door, sign, rail, planter, or chair). Repeated parts need random variation: lights on or off, blind height, AC boxes, color jitter.

## Gameplay layout

The documented `buildGameplay` starts at `s = 90` and lays content with a seeded RNG until `L - 60`:

- 12% × density chance of a **ramp**, plus a 7-collectible airborne parabola (`y = .8 + sin(t·π)·3.4`), 50% chance of a gold collectible at the apex, then `s += 34`.
- About 43% chance of an **obstacle row**, 1–2 obstacles (double-obstacle chance `.25 + stageIndex*.06`, rising difficulty), always leaving at least one open lane.
  - 60% of short obstacles get a 5-collectible arc, rewarding a jump.
  - The open lane gets 4 collectibles as a steering hint.
  - Row spacing `26 + rand*12 - stageIndex*1.5`, tighter later.
- Otherwise a **collectible pattern**: a line of 6, a sine that crosses three lanes (8), or 5 plus one gold.
- Obstacle types come from `region.obstacles`. Each type is `{ hw, hd, h, geo(), move?(), glowAt? }`. Collision is axis-aligned in `s` and `u`, plus height: `P.y < o.h - .15` counts as a hit, so the player can jump short obstacles.
- Stars: collectible ratio ≥ 0.55 and 0 crashes → 3; ≥ 0.35 and at most 1 crash → 2; otherwise 1.

## Performance budget (no traditional mesh LOD)

The documented game did not use multi-LOD meshes. It controlled cost with:

| Lever | Practice |
|---|---|
| Merge | Props become one geometry, one shared material |
| Instancing | Repeats are `InstancedMesh` |
| Fog | Height fog hides things beyond ~300 m, so distant blocks can stay simple |
| Shadow distance | `shadowFar` 110–230 m by tier; clouds and the ground do not cast |
| Particles | A fixed count wraps around the camera, scaled by `particles` (0.45 / 0.75 / 1) |
| Sky | Atmosphere baked once per stage; each frame draws only clouds and stars |
| Post | AO, rays, and depth at half res; toggled by tier; dynamic resolution as a backstop |
| Reflections | Only stages with water or wet ground; scale 0.35–0.75; mid tier may update every other frame |
| Road / terrain | Road section every 2 m; terrain every 5 m (8 m on the low tier) |

**Optional (not documented as used in Pelican World v3):** swap distant blocks for a merged low mesh or `THREE.LOD`; use `BatchedMesh` to cut draw calls further; frustum-cull by `s` segments.
