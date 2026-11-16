# Overview

Use this skill when the user wants a cinematic, offline, **single-file** 3D web game. Pelican World (鹈鹕环游世界) is the worked example: a 3D runner in which a pelican rides a bicycle through six cities. The method is reusable; the game itself is not shipped here.

## The example game (as documented)

The author's notes describe Pelican World v3 as:

- Six stages, in this order: New York → the Great Wall → Japan → Hawaii → Egypt → Paris.
- Everything generated in code: no external models, textures, or audio files.
- One HTML file that is intended to run offline.
- Unprotected build: **971,866 bytes** (0.971866 decimal MB; often called ~0.97 MB). This figure is documented in the notes and was **not** independently re-measured for this package.
- Protected / obfuscated build: **1,674,197 bytes**, which matches a public document measured at https://pelican-world.vercel.app/ on 2026-11-15. See [SIZE.md](../examples/pelican-world/SIZE.md).

The game client remains © 2026 benphant, all rights reserved. Do not copy, vendor, or republish that HTML or its source from this skill.

## Where the "expensive" look comes from

The notes attribute the look to five stacked layers, not to one trick:

| Layer | What it does | Example files in the author's tree |
|---|---|---|
| The sky is the light | A physical atmosphere is baked to a PMREM environment map for all PBR materials; sun color comes from the same model | `gfx/sky.js` |
| Real-time shadows | three.js `SunLight` cascaded shadows (CSM), with PCF rewritten to 12 Vogel-disk samples | `world.js`, `gfx/globals.js` |
| Water reflections | Planar reflection: the scene is drawn mirrored into an RT and fed into PBR indirect specular, so Fresnel comes "for free" | `gfx/water.js` |
| HDR post stack | SSAO → god rays → composite → motion blur → 6-level Bloom → auto-exposure → dirt / flare / CA → ACES/AgX → grade → FXAA + sharpen + grain | `gfx/post.js` |
| Noise as detail | Multi-octave value noise, a three-layer paint (base → grit → light/shadow edges), world-space triplanar micro-normals, vertex AO | `detail/paint.js`, `gfx/surfaces.js`, `util.js` |

Quality in the documented project came from a closed loop: write code → pack one HTML file → headless play and screenshot → criticize the image → change a little → repeat. The author iterated three versions that way.

## Optional vs documented

The original notes mark techniques that were **not** actually used in the project as **【可选进阶】**. In this skill those items are labeled **Optional (not documented as used in Pelican World v3)**. Do not describe them as part of the shipped example.

This skill also does **not** include, and you must not add:

- The third-party reference demo the notes call `reference.html` (described there as a 3.4 MB realistic one-page demo that was studied, not shipped).
- Project-specific domain locks, fingerprint hashes, watermark constants, or obfuscator seeds.

## Honesty

- File size, frame rate, and visual quality of a new game are unknown until measured.
- "Chasing AAA-style visuals in ~1 MB" is an aspirational direction for this method, not a test you can tick off and not a guarantee to the user.
- Software-GPU screenshots prove whether a frame is *correct*, not whether it is *fast*. Call out unverified performance (the notes leave a victory-screen stall on SwiftShader as unsolved).
