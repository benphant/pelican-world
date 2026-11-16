# Third-party notices

This package distributes **documentation and an agent skill only**. It does
**not** include the Pelican World game, the game's source code, any built
HTML of the game, or the third-party reference demo mentioned in the
original methodology notes as `reference.html`. Do not add those files to
a public copy of this skill unless you have a separate, explicit license
to do so.

The playable game is published separately, under its own terms, at
[https://pelican-world.vercel.app/](https://pelican-world.vercel.app/).
That build is **not** licensed under Apache-2.0.

## Software this tutorial tells you to use

If you follow the method and produce your own game, you will typically
depend on third-party tools and libraries. They are **not** shipped in
this package. You must obtain them yourself and follow their licenses.

### three.js

- License: MIT License
- Copyright: three.js authors (the methodology notes record a notice of
  the form `three.js (MIT License, (c) 2010-2026 three.js authors)`)
- Homepage: [https://threejs.org/](https://threejs.org/)
- Source: [https://github.com/mrdoob/three.js](https://github.com/mrdoob/three.js)

The documented build inlines three.js into a single HTML file. When you
do that, keep the MIT license notice in the bundle. The methodology
recommends esbuild `legalComments: 'inline'` plus a `/*! ... */` banner
so minification does not drop required notices.

three.js also includes examples such as `SunLight` (cascaded shadow
maps) and `BufferGeometryUtils`. Those files carry the same project
license terms unless a specific file says otherwise. Import only what
you use.

### Other tools named in the method

These may appear in a project you create. This package does not vendor
them.

| Tool | Role in the method | License (check upstream) |
|---|---|---|
| [esbuild](https://esbuild.github.io/) | Bundle and minify the game into one HTML file | MIT |
| [Playwright](https://playwright.dev/) / playwright-core | Headless screenshot and logic tests | Apache-2.0 |
| Chromium / Google Chrome | Test browser, including software (SwiftShader) rendering | See the browser vendor's terms |
| [javascript-obfuscator](https://github.com/javascript-obfuscator/javascript-obfuscator) | Optional, only if you later choose to obfuscate a public build | BSD-2-Clause (verify the version you use) |

## Techniques and attributions described in the tutorial

The skill documents implementation recipes inspired by well-known
graphics practice. Names below are attributions, not a claim that this
package redistributes those authors' code.

- Atmospheric scattering in the style of Nishita, with Rayleigh and Mie
  terms and ozone absorption.
- Cornette–Shanks and Henyey–Greenstein phase functions, as commonly
  described in the graphics literature.
- Oblique near-plane clipping for planar reflections (Lengyel).
- ACES tone mapping using Stephen Hill's RRT+ODT fit (the documented
  implementation scales the input by 1.3).
- AgX tone mapping (Rec.2020, log2 polynomial fit), as commonly
  described in publicly documented AgX implementations.
- Karis average used on the first Bloom downsample to reduce fireflies.
- FXAA as a final anti-aliasing pass.
- Interleaved gradient noise for rotating SSAO samples.

If your own code copies a third-party shader or implementation, keep
that author's license and notice. This tutorial's Apache-2.0 terms do
not replace those obligations.

## Scope reminder

Landmark-like props in a procedural city (for example a tower or a
statue) are artistic reconstructions in a game, not official or
licensed depictions. Do not describe them as endorsed by any trademark
or rights holder.
