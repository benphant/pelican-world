---
name: pelican-world
description: "Build a playable, offline, single-file HTML 3D game with procedural models, textures, and audio, using a build-then-screenshot review loop, physical lighting, PBR materials, derived animation, and quality tiers. Use when the user wants a cinematic single-file web game, a ~1 MB-class HTML game, or to apply the Pelican World method. Not for shipping or redistributing the Pelican World game client (separately licensed). Not for CDN- or asset-pack-based games, native engines, or dashboards."
license: Apache-2.0
metadata:
  version: "0.1.0"
  author: benphant
---

# Pelican World

Help the user build a **playable, offline, single-file** 3D web game: one HTML file, no CDN, no external models, textures, fonts, or audio files. Visual quality comes from procedural generation plus a disciplined screenshot review loop, not from shipping large media.

Pelican World (鹈鹕环游世界) is the **worked example** behind this skill. The game client is **not** in this package and remains all-rights-reserved. If the user wants to play it, send them to https://pelican-world.vercel.app/ — do not download or redistribute that HTML as if it were part of the skill.

This skill teaches a method. It does **not** guarantee a file size, a frame rate, or "AAA" quality. The author's documented unprotected v3 build is 971,866 bytes (~0.97 decimal MB); that figure was not independently re-measured here. The public playable build was measured at 1,674,197 bytes on 2026-11-15. See [SIZE.md](examples/pelican-world/SIZE.md). Aspirational marketing copy such as "chasing AAA-style visuals in ~1 MB" is a direction, not an acceptance test.

## When to read what

Read this file first and follow the workflow below. Open a reference only when its topic is in scope:

| Topic | Open |
|---|---|
| Constraints, example game, honesty rules | [Overview](references/overview.md) |
| Folders, esbuild inlining, quality tiers | [Architecture and build](references/architecture-and-build.md) |
| Sky, shadows, fog, water, post stack | [Lighting](references/lighting.md) |
| Damping, locomotion, IK, camera | [Animation](references/animation.md) |
| Noise, layered canvases, PBR, triplanar | [Materials](references/materials.md) |
| Regions, path coordinates, props, layout | [World building](references/world-building.md) |
| Web Audio music and SFX | [Audio](references/audio.md) |
| Tokens, glass panels, HUD, responsive UI | [UI](references/ui.md) |
| Playwright loop, pitfalls, software GPU | [Testing and iteration](references/testing-and-iteration.md) |
| Attribution, headers, hosting (generic) | [Publishing](references/publishing.md) |
| Screenshot, shipping, and texture checks | [Checklists](references/checklists.md) |
| Starting prompt for a new game | [Prompt template](references/prompt-template.md) |
| Documented vs measured byte counts | [SIZE.md](examples/pelican-world/SIZE.md) |

Items marked **Optional (not documented as used in Pelican World v3)** were not part of the author's shipped method. Do not present them as completed work.

## Hard constraints

Unless the user explicitly relaxes a constraint, treat these as acceptance tests:

1. **One HTML file** is the deliverable. It must run completely offline (including in mainland China without a VPN). Bundle three.js with esbuild; use a system font stack only; generate all models, textures, and audio in code, or inline tiny data URLs created at build time (for example, level-select thumbnails).
2. **Keep a playable file on disk after every step.** Rebuild often so an interruption never leaves the user with only a broken tree.
3. **Preserve third-party license notices** in the bundle, especially three.js MIT text. Use esbuild `legalComments: 'inline'` and `/*! ... */` banners so minification does not drop them. See [THIRD_PARTY_NOTICES.md](../THIRD_PARTY_NOTICES.md).
4. **Do not load `reference.html` or any third-party demo into the project, and do not redistribute it.** Teach reusable techniques only.
5. **Do not copy project-specific host locks, fingerprints, or obfuscator seeds** from any prior notes. Publishing guidance in this skill is generic.

## Workflow

Every increment ends in a state that is **playable and screenshot-able**.

1. Agree on theme, playable loop, and author credit. Put the credit in `<title>`, `meta author`, the end card, the console, and a `/*! ... */` legal comment.
2. Scaffold `template.html`, `src/main.js`, and `build.mjs`. Get a rotating cube in a single HTML file. Run the offline network assertion (zero non-`file:` / `data:` / `blob:` requests).
3. Follow the recommended build order in the table below. After each step: rebuild, screenshot, write down what looks cheap, change one or two things, repeat.
4. Report honestly: file paths, **on-disk byte length of the final HTML**, contact sheets, and known issues. If you did not verify something (size, GPU performance, a visual claim), say so. Do not claim AAA quality or a target file size as fact.

### Recommended build order

| Step | Work | Done when | Reference |
|---|---|---|---|
| 1 | Scaffold + inline three.js + a moving primitive | Single file opens; offline request count is 0 | [Architecture](references/architecture-and-build.md) |
| 2 | Headless test hook: `window.__game`, wait on **frame count** (not a fixed timeout), screenshot | Non-black frames | [Testing](references/testing-and-iteration.md) |
| 3 | Road frame `(s, u)`, terrain, road mesh, player, follow camera with `damp` | A bend in the road is visible (check normals) | [World](references/world-building.md), [Animation](references/animation.md) |
| 4 | Lanes, obstacles, collectibles, jump, ramps, collision, score, state machine | An auto-play script can finish a full stage | [World](references/world-building.md) |
| 5 | Hero assembled from primitives + motion derived from physics (wheels, pedals, IK, secondary motion) | Close-up: contacts stick; body leans in turns | [Animation](references/animation.md) |
| 6 | Physical sky → PMREM IBL → cascaded sun shadows → sky-driven height fog | Horizon matches the sky; shadows read clearly | [Lighting](references/lighting.md) |
| 7 | HDR post: Bloom, auto-exposure, ACES/AgX, FXAA; then SSAO, god rays, motion blur, dirt, flares, grade | Each pass is compared in screenshots; quality tiers can disable passes | [Lighting](references/lighting.md) |
| 8 | Planar-reflected water in the PBR pipeline; optional wet ground | Reflections sit on the plane; no feedback loop | [Lighting](references/lighting.md) |
| 9 | Layered canvases, normals, triplanar micro-detail, vertex AO | Close-ups are not large flat colors or regular grids | [Materials](references/materials.md) |
| 10 | Data-driven regions; finish one flagship stage before cloning the pattern | 3–4 fixed cameras per stage, collected into a contact sheet | [World](references/world-building.md) |
| 11 | High-detail generators (real facade hierarchy); instance repeats | Density holds up at 5–30 m | [World](references/world-building.md) |
| 12 | Audio scheduler, per-stage music data, SFX; pause flags derived from game state | Music counters keep rising after restart, title, and stage changes | [Audio](references/audio.md) |
| 13 | Design tokens, glass panels, title / stage select / HUD / pause / results; responsive + touch | Desktop and phone screenshots, no overflow | [UI](references/ui.md) |
| 14 | Quality tiers, dynamic resolution, no-WebGL2 fallback | `?q=0..3` screenshots; an FPS overlay exists | [Architecture](references/architecture-and-build.md) |
| 15 | Only at release: attribution, generic hosting checks, optional separate public artifact | Production URL: 0 page errors, 0 external requests | [Publishing](references/publishing.md), [Checklists](references/checklists.md) |

## Authoring rules

- Prefer **seeded** randomness so two builds can be compared.
- Measure the **real artifact**: `Buffer.byteLength` of the file you wrote, frame counters, note counters, network request counts. Do not estimate.
- Animation and camera smoothing use frame-rate-independent damping: `damp(a, b, lambda, dt) = lerp(a, b, 1 - Math.exp(-lambda * dt))`. Never use `x += (target - x) * 0.1`.
- Wait for **frames**, not wall-clock time, before screenshots. Software renderers may run at 1–3 FPS and take tens of seconds to compile shaders.
- Build **one flagship stage to a high standard**, then copy the pattern. Do not spread weak detail across every stage first.
- When the user says the image looks cheap, turn that into a concrete list (which screen, which surface, what is missing) before editing.
- In compact, single-line source, use `/* */` comments, not `//`, so a trailing `}` is not commented out.

## Output

When you finish a step or a session, report:

- Paths of the playable HTML and the source you changed.
- Byte length of the playable HTML (the file on disk).
- Contact sheets or representative screenshots, using a **new filename** each time you regenerate (do not overwrite a path the review tools may have cached).
- Known issues, including anything you could not verify (especially performance on a real GPU).

If the user asks to publish, follow [Publishing](references/publishing.md). Do not invent a GitHub URL, a file size, or a quality claim.
