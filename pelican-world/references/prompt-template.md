# Prompt template

Copy this into a new agent session after the Pelican World skill is available. Replace the bracketed fields. The skill's reference files are the technical source of truth; do not point the agent at a private methodology dump or at a third-party `reference.html`.

```
You are a graphics programmer and game designer. Build a 3D web game with the Pelican World skill. Follow pelican-world/SKILL.md and open its references as that file directs.

[Theme]: for example, "a corgi driving a toy train through four seasons"
[Play]: for example, "three-lane runner: change lanes, jump, collect bones, avoid obstacles, 60–90 s per stage"
[Stages]: for example, "spring blossom / summer sea / autumn wood / winter snow", each with one landmark and its own time of day and weather
[Credit]: for example "Your Name", written into <title>, meta author, the end card, the console, and a /*! */ legal comment

Hard constraints:
1. The only deliverable is one .html file that runs fully offline (including in mainland China, with no CDN, no Google Fonts, and no external images).
   Install three.js from npm and inline it with esbuild; use a system font stack; generate all textures, models, and music in code.
2. Keep source in modules under src/ (main, world, regions, rider or hero, audio, util,
   gfx/{sky,post,water,surfaces,globals,quality}, detail/), and have build.mjs inline the
   bundle into template.html. Rebuild after every step so a playable file is always on disk.
3. The picture should be realistic, using the techniques in the lighting and materials references:
   - Physical atmosphere (Rayleigh + Mie + ozone), baked to PMREM as scene.environment;
     sun color and fog color from the same model.
   - three/examples/jsm/lights/SunLight cascaded shadows, PCF rewritten to 12 Vogel-disk samples.
   - Exponential height fog driven by sky colors (replace the fog ShaderChunks).
   - Planar-reflected water (Lengyel oblique clip) fed into MeshPhysicalMaterial indirect specular;
     optional wet-ground puddles.
   - HalfFloat HDR target + a custom post stack: half-res SSAO (ambient only) → screen-space god rays
     → motion blur → 6-level Bloom (Karis average on the first downsample) → auto-exposure (log-average
     luminance) → lens dirt → sun flare → CA → ACES/AgX mix → split-tone grade → vignette
     → FXAA + sharpen + film grain + dither.
   - Four quality tiers, auto-selected from the GPU name, plus dynamic resolution;
     if WebGL2 is missing, fall back to no custom post.
4. Materials: "three-layer paint" (base shapes → multi-octave noise and grime → lit edges and contact
   shadows), outputting aligned color, height→normal, roughness, and emissive canvases;
   merged geometry uses vertex colors + world-space triplanar micro-detail (rain streaks, splash-zone
   dirt, roughness jitter) + a vertex AO bake; instance anything that repeats.
5. Motion: derive animation from physical quantities (wheel rate = v/r, body bob from crank phase),
   two-bone IK so limbs stay on contact points, secondary motion with a spring (for example k=160, c=6),
   and frame-rate-independent damp(a, b, λ, dt) = lerp(a, b, 1 - exp(-λ·dt)).
   Follow camera: position λ=7, look λ=12, FOV grows with speed, land and crash have camera shake.
6. Audio: Web Audio procedural music (look-ahead scheduler + per-stage scale / chords / instrument)
   and synthesized SFX. Derive the pause flag from game state every frame so music continues after
   retry, title, and stage changes.
7. UI: a dark token system (ink / bone / one accent / hairlines / tabular numbers), three-layer glass
   panels, title + stage select (inlined thumbs, stars, best score) + stage card + countdown + HUD
   + pause + results + victory; localStorage wrapped in try/catch; responsive layout and touch controls.
8. Tests: playwright-core + SwiftShader (--use-gl=angle --use-angle=swiftshader --enable-unsafe-swiftshader)
   in an offline context:
   (a) fixed cameras per stage (wait on a frame counter, not a fixed timeout),
   (b) an auto-play script finishes every stage,
   (c) a music counter covers every flow,
   (d) desktop and phone UI screenshots,
   (e) assert 0 pageerror and 0 external requests.
   Every round, look at the screenshots, list what looks cheap, and only then edit.
9. At each report, give: file paths, byte length of the playable HTML, contact sheets, and known issues.
   If something is uncertain, say so. Do not claim work you did not do, a file size you did not measure,
   or AAA quality.

This skill does not include the Pelican World game or any third-party reference demo. Do not download
or vendor https://pelican-world.vercel.app/ into the project.
```
