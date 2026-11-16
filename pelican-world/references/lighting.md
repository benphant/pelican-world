# Lighting

Core idea, taken from studying a realistic one-page demo (that file is **not** redistributed here): **the sky is a shader, and that same sky lights the scene; sun shadows are real-time; water redraws the scene inverted; post (contact occlusion, shafts, dirt, auto-exposure, motion blur) is what makes the image feel finished.**

Renderer defaults in the documented game: `toneMapping = NoToneMapping` (tone mapping happens in the grade pass), `autoClear = false`, `antialias: false` (MSAA render targets plus FXAA), `shadowMap.type = PCFShadowMap`.

## One frame, in order

```
renderFrame():
  1. Planar reflection pass    world.refl.render()
     — mirror the camera, draw the scene into a half-res HDR RT
       (only in regions that have water or wet ground)
  2. Shadow pass               shadowMap.needsUpdate
     — SunLight, two CSM cascades, possibly every other frame
  3. Main scene                render(scene) → HDR RT
     — HalfFloat + DepthTexture; MSAA 4x on high tiers
  4. post.run():
     a. Linear depth (half res)
     b. SSAO (half res) + bilateral blur H/V
     c. Screen-space god rays (half res; only if the sun is on screen)
     d. Composite: AO darkens ambient only, then add rays
     e. Camera motion blur (high tiers)
     f. Bloom pyramid: 6 downsamples + 5 upsamples
     g. Auto-exposure (1×1 ping-pong; also measures sun visibility)
     h. Grade: CA → bloom → dirt → sun flare → exposure
        → ACES/AgX mix → split-toning → contrast → lift
        → saturation → vignette → sRGB
     i. Final: FXAA + sharpen + film grain + dither → screen
```

## Physical sky

**Model:** Nishita-style single scattering (Rayleigh + Mie + ozone absorption). Ray-march between planet radius `6371e3` and atmosphere top `6471e3`: 24 steps on the view ray, 8 toward the sun, including the planet's own shadow.

```glsl
vec3 kR = vec3( 5.8e-6, 13.5e-6, 33.1e-6 ) * uRay;
float kM = 21e-6 * uMie;
float pR = 3.0 / ( 16.0 * PI ) * ( 1.0 + mu * mu );              // Rayleigh
float pM = 3.0 / ( 8.0 * PI ) * ( ( 1.0 - gg ) * ( mu * mu + 1.0 ) )
         / ( pow( 1.0 + gg - 2.0 * mu * g, 1.5 ) * ( 2.0 + gg ) ); // Cornette-Shanks Mie
float dR = exp( - h / 8000.0 ) * stp, dM = exp( - h / 1200.0 ) * stp; // 8 km / 1.2 km
vec3 att = exp( - ( kM * 1.11 * ( odM + jM ) + kR * ( odR + jR ) + kO * ( odO + jO ) ) );
```

**Cheap multiple scattering:** single scattering goes green near the horizon; a real sky is milky. Mix toward a bluish white by luminance:

```glsl
float l = dot( c, vec3( 0.2126, 0.7152, 0.0722 ) );
float w = 0.55 * exp( - max( d.y, 0.0 ) * 5.0 ) * smoothstep( 0.03, 0.45, uSunDir.y );
c = mix( c, l * vec3( 0.93, 0.99, 1.12 ), w );
```

**Performance:** bake the atmosphere once when the region changes, into a 128² HalfFloat cube map (the sun does not move during a run). Each frame a cheap shader layers, on that cube:

- Sun disk: a `sunSize`-scaled smoothstep disk, `col += uSunCol * disk * 60.0 * (1.0 - uEnv)`.
- Clouds: screen-space fbm with domain warp; a second sample offset toward the sun for a lit relief; Henyey–Greenstein silver lining; distant clouds fade into fog.
- Stars, moon, and night-time urban glow.
- Below the horizon: ground albedo for IBL, haze color for the visible sky.

**The same model on the CPU** (`atmosJS`, `sunTransmittance`):

- Sun color = transmittance of sunlight to the ground (32 steps). Dusk goes orange; noon goes white.
- Ambient = average of zenith and side sky.
- Fog color = samples of "horizon away from the sun", "horizon toward the sun", and zenith, so distant terrain meets the sky (aerial perspective).

**IBL:** run the sky shader with `uEnv = 1` so the sun disk is omitted (the directional light already represents the sun). `PMREMGenerator.fromScene()` becomes `scene.environment`. Every `MeshStandardMaterial` / `MeshPhysicalMaterial` then receives that sky.

**Per-region parameters** (Paris "golden hour" as documented):

```js
sky: { sunDir: V(.42, .19, -.89), ray: 1.1, mie: 2.2, mieG: .82, sunI: 3.9,
       cloudCov: .36, cloudScale: .9, groundAlbedo: '#8a7a5a',
       fogDensity: .00145, fogFalloff: .011, fogTint: '#ffe2c0',
       sunSize: 1.7, exposure: 1.05 },
grade: { shadowTint: [.84, .95, 1.16], highTint: [1.14, .99, .82],
         sat: 1.08, contrast: .24, key: .16, rays: 1.75, rayCol: [1, .74, .46] },
```

- `sunDir.y ≈ 0.19` is about 11° elevation: long shadows, stronger shafts. Place the sun ahead of the player (−z) for backlight.
- Larger `mie` and `mieG` thicken the glow around the sun. `ray` controls how blue the sky is.
- **Night** (New York in the example): put the sun below the horizon (`sunDir.y = -.11`), use `moonLight` as the directional light (color `#a9bcff`, intensity 0.6), add `nightCol` / `glowCol` city pollution, and enable `stars` and `moon`.

## Directional light and cascaded shadows

```js
this.sun = new SunLight('#ffffff', 3); // three/examples/jsm/lights/SunLight.js (CSM)
this.sun.castShadow = true;
this.sun.shadow.mapSize.set(2048, 2048);
this.sun.shadow.camera.far = 190;
this.sun.shadow.camera.near = .5;
this.sun.shadow.bias = -0.0003;
this.sun.shadow.normalBias = .035;
this.sun.shadow.radius = 2.2;
```

- Two cascades fitted to the frustum and snapped to texels to reduce edge swimming. Documented `shadowFar` per tier: 110, 150, 190, 230. Map size: 1024 to 4096.
- Intensity follows the atmosphere: `sun.intensity = sunI * pow(max(r,g,b), .7)`, then normalize the color, so a low sun is both dimmer and warmer.
- Softer shadows: three.js default PCF uses 5 taps. The documented game rewrites `ShaderChunk.shadowmap_pars_fragment` to 12 Vogel-disk samples:

```js
'shadow = 0.0; for ( int i = 0; i < 12; i ++ ) shadow += texture( shadowMap, vec3( shadowCoord.xy + vogelDiskSample( i, 12, phi ) * radius, shadowCoord.z ) ); shadow *= ( 1.0 / 12.0 );'
```

- Tuning: `normalBias` in 0.02–0.05 removes most shadow acne. Too negative a `bias` separates the shadow from the caster (Peter Panning).

## Fog and aerial perspective

Replace three.js `fog_*` ShaderChunks so every material, including sprites, picks the new fog up automatically.

```glsl
vec3 slFogColor( vec3 rd ) {
  // Directional fog color: anti-sun → sun side, lerp toward zenith when looking up, add Mie glow
  float ts = clamp( dot( normalize( rd.xz ), sxz ) * 0.5 + 0.5, 0.0, 1.0 );
  vec3 c = mix( uFogAway, uFogToward, ts * ts * ts );
  c = mix( c, uFogZenith, smoothstep( 0.0, 0.6, rd.y ) * 0.6 );
  float mu = max( dot( rd, uFogSunDir ), 0.0 );
  c += uFogSunCol * ( pow( mu, 8.0 ) * 0.35 + pow( mu, 48.0 ) * 0.6 ) * uFogParams.w;
  return c;
}
float slFogAmount( vec3 off ) {
  // Analytic integral of exponential height fog: denser near the ground
  float dist = length( off ); vec3 rd = off / max( dist, 1e-4 );
  float k = uFogParams.y; float h0 = cameraPosition.y - uFogParams.z; float dy = rd.y * k * dist;
  float od = uFogParams.x * exp( - k * max( h0, -20.0 ) ) * dist
           * ( abs( dy ) > 1e-3 ? ( 1.0 - exp( - dy ) ) / dy : 1.0 - 0.5 * dy );
  return 1.0 - exp( - max( od, 0.0 ) );
}
```

- Fog uniforms live in a **shared Float32Array**. `UniformsUtils.clone()` copies a typed array by reference, so updating `FOG.away.set(...)` once updates every material.
- Additive sprites (lamp glows) should define `SL_FOG_ADDITIVE` so fog only attenuates, and does not add fog color. Premultiplied-alpha water should define `SL_FOG_PREMUL`.
- Typical density: `fogDensity` 0.0015–0.0034, `fogFalloff` 0.009–0.012. Too much fog washes the image to grey. Paris settled at 0.00145.

## Planar-reflected water and Fresnel

The method is: **flip the scene and draw it again in the water.**

1. Mirror camera position and orientation across the plane `y = planeY`.
2. Use a **Lengyel oblique near plane** so geometry below the water does not appear in the reflection.

```js
this._plane.setFromNormalAndCoplanarPoint(n, P).applyMatrix4(rc.matrixWorldInverse);
const cp = this._clip.set(plane.normal.x, plane.normal.y, plane.normal.z, plane.constant);
const pm = rc.projectionMatrix.elements;
q.x = (Math.sign(cp.x) + pm[8]) / pm[0];
q.y = (Math.sign(cp.y) + pm[9]) / pm[5];
q.z = -1; q.w = (1 + pm[10]) / pm[14];
cp.multiplyScalar(2 / cp.dot(q));
pm[2] = cp.x; pm[6] = cp.y; pm[10] = cp.z + 1 - .003; pm[14] = cp.w;
```

3. Render into a half-res HalfFloat RT with mipmaps; blur the reflection by sampling a higher mip. During the reflection pass:
   - Do not update shadows.
   - Keep water on `WATER_LAYER = 1`; the reflection camera draws only layer 0.
   - **Bind a 1×1 dummy texture in place of the reflection map** for that pass. Otherwise wet ground would sample the RT it is also writing (a feedback loop). The notes record this as a real bug (flicker or GL errors).

4. **Feed the result into PBR, do not hand-blend.** On a `MeshPhysicalMaterial` (`ior: 1.333`), after `lights_fragment_maps`, replace indirect specular `radiance` with the planar sample. Fresnel (brighter at grazing angles), roughness (blurrier when rough, `lod = roughness * 9`), and sun highlights then come from three.js's BRDF:

```glsl
#if defined( RE_IndirectSpecular )
{ float dist = length( cameraPosition - vWW );
  vec2 tilt = wN.xz * vec2( 0.6, 1.0 ) * 0.22 / ( 1.0 + dist * 0.05 );
  radiance = slPlanar( vWW, tilt, material.roughness * 9.0 + wIce * 2.0, radiance ); }
#endif
```

Other documented water details:

- **Ripple normals:** 48 integer-frequency sine waves into a 256² tileable map (integer frequencies stay seamless). Three scales and directions; fade with distance.
- **Waves:** Gerstner waves in the vertex shader, attenuated near shore by `aDepth` (water height minus terrain height).
- **Color / alpha:** `absorb = 1 - exp(-depth * clarity)`, lerp shallow to deep color; alpha rises with depth.
- **Shore foam:** appears when depth is under 1.6 m, with time-scrolling bands.
- **Ice** (moat on the Great Wall stage): a noise mask flattens normal perturbation to 5%, roughness 0.32, a blue-white tint.
- **Wet ground** (New York / Paris `wet: { puddles: .48 / .22 }`): the same reflection RT, a noise puddle mask `gPud`:

```glsl
diffuseColor.rgb *= mix( 0.62, 0.32, gPud );                 // wet surfaces are darker
roughnessFactor = mix( roughnessFactor * 0.55, 0.03, gPud );  // puddles are mirrors
normal = mix( normal, up, gPud );                             // puddles are flat
// Non-puddle wet areas: higher mips, two offset samples, stretched reflections
```

## SSAO

- Compute on half-res linear depth. Reconstruct normals from depth; pick the smaller of the left/right and up/down derivatives so object silhouettes do not break.
- Spiral-sample `AO_TAPS` points (8 / 8 / 12 / 16 by tier). Rotate the kernel with interleaved gradient noise `ign()` to kill banding.
- Radius grows with distance: `R = uRadius * (0.6 + 0.4 * clamp(z/12, 0, 1))`. Fade out between 80–160 m.
- 7-tap depth-aware **bilateral blur**, once horizontal and once vertical.
- **AO should darken ambient only.** At composite time, weaken AO on sunlit pixels, or lit faces get a dirty halo:

```glsl
float lit = smoothstep(0.16, 0.85, luma(col) * slExposure());
float a = mix(1.0, ao, uAoStrength * (1.0 - 0.75 * lit));
col *= mix(vec3(a), vec3(1.0), vec3(0.0, 0.03, 0.07)); // crush blue a little less: cool shadows
```

- Upsample with 4-tap bilinear weighted by depth similarity so AO does not leak past foreground silhouettes.

## Screen-space god rays

Run only when the sun is on screen (or within 35% of the screen edge). From each pixel, walk `RAY_STEPS` (24–56) steps toward the sun's screen position, accumulating only "sky" pixels (depth greater than `far × 0.97`), decaying 0.965 per step, then `exp(-r*1.6)` by distance to the sun. Multiply by `rayCol × rays` and add. Paris dusk used `rays: 1.75, rayCol: [1, .74, .46]`.

```glsl
for (int i = 0; i < RAY_STEPS; i++) { uv += st;
  float sky = step(uFar * 0.97, texture2D(tDepthHalf, uv).r);
  vec3 c = min(texture2D(tColor, uv).rgb, vec3(uClamp)); // clamp so the sun disk itself does not blow out
  acc += c * sky * illum; illum *= 0.965; }
```

## Motion blur

Reconstruct world position from depth, multiply by the previous ViewProj to get screen velocity, and sample 9 times along that vector.

- Shutter: `uShutter = min(.5, 1/120/dt)`, independent of frame rate.
- Velocity cap: 0.025 of screen width. Skip if under 0.75 px (zero cost when still).
- Disable on a camera cut (`opts.cut`), or the whole frame smears.

## Bloom and auto-exposure

- **Bloom pyramid:** 6 downsamples. The first uses a 13-tap kernel with a Karis average (`1/(1+luma*0.08)`) to kill fireflies; then 5 levels of 3×3 tent upsample, `uMix = .62`. Default `bloom: .07`, night `.1`.
- **Auto-exposure:** from the smallest pyramid level, a 7×5 grid of **log-average luminance** with a center weight. Store in a 1×1 ping-pong RT, smoothed with `tau = 1.3 s`. The same pass samples 12 points around the sun to see whether it is occluded (flare visibility).

```glsl
float slExposure(){ float a = texture2D(tAdapt, vec2(0.5)).r;
  return uExposure * mix(1.0, clamp(pow(uKey / max(a, 1e-4), /* … */), uMinE, uMaxE), uAuto); }
```

Region parameters: `key` (target mid-grey; day 0.15–0.16, night 0.11) and `minE` / `maxE` (exposure range; night `.6/6`).

## Lens dirt, sun flare, chromatic aberration

- **Dirt** is generated once at startup into a 512×288 RT: 70 soft random blobs, two fbm grime layers, vertical scratches, stronger toward the edges. At use: `col += bloom * dirt * uDirt` — dirt is only visible where bright light hits it, as on a real lens. Default `dirt: .12`.
- **Flare:** a center glow, two `cos` starbursts (`pow(abs(cos(ang*3+.4)),24) + .6*pow(abs(cos(ang*4-1.1)),40)`), four colored ghosts along the sun–center line, plus a halo; multiplied by measured sun visibility so the flare dies when a building covers the sun. Default `flare: .14`.
- **CA:** offset R and B by the square of the distance to the center, `ca: .0013`.

## Tone mapping and grade (ACES / AgX mix)

```glsl
col *= slExposure();
col = mix(aces(col), agx(col), uAgxMix);                       // default agx: .25 — ACES contrast + AgX highlight desat
float l = luma(col);
col *= mix(uShadowTint, uHighTint, smoothstep(0.03, 0.55, l)); // split tone: cool shadows, warm highlights
col = mix(col, col * col * (3.0 - 2.0 * col), uContrast);      // S-curve contrast
col = col * (1.0 - uLift * 1.2) + uLift;                       // slightly blue lift
col = mix(vec3(luma(col)), col, uSaturation);
col *= 1.0 - uVignette * vr * vr * (3.0 - 2.0 * vr);
```

- ACES uses Stephen Hill's RRT+ODT fit (input ×1.3). AgX converts to Rec.2020, applies a 6th-order polynomial in log2, and converts back.
- Default grade (`DEFAULT_GRADE`): `sat 1.05, contrast .18, shadowTint [.86,1,1.1], highTint [1.06,1,.92], lift [.006,.0085,.011], vignette .3`. Each region overrides only a few fields.
- The grade pass writes luminance into alpha for the FXAA pass.

## FXAA, sharpen, grain

The last pass, in order:

- FXAA on the alpha luminance, contrast threshold `max(0.0312, lMax * 0.10)`.
- Unsharp sharpen off edges, `sharpen .28`.
- Time-varying film grain (`grain .022`), weaker in deep shadows and highlights.
- ±1/255 triangular dither to break banding in shadows and sky.

## Lighting rules of thumb

1. **Set the sun direction first.** Low sun (10–20°) plus backlight is the fastest path to a cinematic frame.
2. **Prefer thin fog.** Thick fog blows out both god rays and the sky.
3. Emissive surfaces (windows, neon, shop interiors) must stay **restrained in HDR**. v3 Paris windows started as solid orange rectangles. The fix: facade `emissiveIntensity` 0.8, radial gradients on the emissive map, muntins and curtains as occluders, and only about 14% of windows lit. New York used about 24% lit windows.
4. AO on a lit face looks dirty; always fade it by how lit the pixel is.
5. Night readability needs **fill**: New York used `fill: .42`, `envI: 1.6`, and an additive sidewalk pool in front of each shop window (opacity 0.32).
