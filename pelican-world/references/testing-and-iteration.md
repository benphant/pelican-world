# Testing and iteration

Visual quality in the documented project came from a tight loop: build → headless play and screenshot → criticize the contact sheet → change a little → rebuild. Logic and audio have their own measurable hooks.

## Playwright launch (software GPU)

```js
const browser = await chromium.launch({
  executablePath: '/usr/bin/google-chrome',
  args: [
    '--use-gl=angle',
    '--use-angle=swiftshader',
    '--enable-unsafe-swiftshader',
    '--ignore-gpu-blocklist',
    '--autoplay-policy=no-user-gesture-required' // needed if you assert music
  ]
});
const ctx = await browser.newContext({ viewport: { width: 1280, height: 720 }, offline: true });
page.on('request', r => { if (!/^(file|data|blob):/.test(r.url())) net.push(r.url()); });
page.on('pageerror', e => logs.push('PAGEERROR: ' + e.message));
```

Assert: `net.length === 0` and no `pageerror`. Adapt `executablePath` to the machine you actually have; do not assume a single Chrome path.

## Wait for frames, not for a timeout

Under SwiftShader the game may run at 1–3 FPS, and the first compile of a new shader variant can take tens of seconds. A fixed `waitForTimeout` often captures the previous frame, or a black frame.

```js
const f0 = await page.evaluate(() => __game.frames());
const tw = Date.now();
while ((await page.evaluate(() => __game.frames())) < f0 + 10 && Date.now() - tw < 120000) {
  await page.waitForTimeout(400);
}
await page.screenshot({ path, timeout: 120000 });
// backdrop-filter is heavy in software; 120 s screenshot timeout was necessary in the notes
```

After a camera cut, call the game's `cut()` hook (in the example, `__game.cut()`) so post skips motion blur and exposure converges.

When you regenerate a contact sheet, **use a new filename** (`sheet_v2.png`, `sheet_v3.png`, …). Some review tools cache images by path; overwriting the same file can show a stale picture.

## Auto-play for layout and completion

The documented `logic.mjs` ran `g.step(1/60, 1)` in page context, switched to an empty lane for tall obstacles, and jumped 3.5 m early for short ones. Use something equivalent to prove every stage can be finished (`state: 'finish'`, distance `s === L`) and to count collectibles, obstacles, and crashes.

Lock the quality tier during software-GPU runs (`?q=N` or `__game.setQuality()`). Otherwise dynamic resolution will fall to the floor at 1–3 FPS and you will not be testing the intended picture.

## Pitfalls recorded in Pelican World v3

| Id | Symptom | Cause | Fix / lesson |
|---|---|---|---|
| 9.1 | A whole road is invisible | Hand-built road triangles had reversed winding; normals pointed down and were back-face culled | Indices `a, a+1, a+2, a+1, a+3, a+2`. **Check normals on every hand-built mesh**: after `computeVertexNormals()`, `normal.getY(0)` should be positive. The river strip in the example auto-corrects this way |
| 9.2 | A retaining wall is invisible | `pathStrip(u0, u1)` winding depends on which of `u0`, `u1` is larger | Swap the arguments. When a parameter chooses facing, confirm **both** sides in screenshots |
| 9.3 | No music after retry | A pause flag was cleared on only one exit | See [Audio](audio.md): derive the flag from state every frame, and clear it on every entry |
| 9.4 | Reflection RT errors or flicker | Wet-ground materials sampled the RT being written in the reflection pass | Bind a 1×1 dummy texture during that pass |
| 9.5 | A grid on the ground | Ground-normal noise included a regular `sin(u)·sin(v)` term, magnified at grazing angles | Remove it; use fbm plus a ridge modulator. **Do not put regular trigonometric terms in procedural textures** |
| 9.6 | Shop windows blow out to white | Emissive maps were flat bright color; HDR plus auto-exposure pegged them | Emissive = albedo × light color, and lower the intensity. **Emission needs albedo detail** |
| 9.7 | Lit windows look like orange rectangles | Emissive too strong, and each pane was a flat fill | Intensity 0.8, radial gradients, muntins and curtains; only ~14% of panes lit (Paris) |
| 9.8 | Screenshot is stale or black | 1–3 FPS and long shader compiles; a fixed wait was too short | Wait for 10 more frames; 120 s screenshot timeout; call `cut()` after a camera change |
| 9.9 | Screenshots are extremely slow or time out | `backdrop-filter` is costly in software | Raise the timeout; real-GPU players are mostly unaffected |
| 9.10 | Build succeeds, game is a white screen | A `//` comment in a one-line function also commented out the closing `}` | In compacted one-line code use only `/* */`; after every build, run a headless smoke test and watch `pageerror` |
| 9.11 | The build script prints the wrong size | It was not printing the final file | Print `Buffer.byteLength` of what you wrote. **Measure the real artifact** |
| 9.12 | The victory screen stops advancing frames under SwiftShader | Unsolved in the notes (fireworks, CSS, and state were ruled out; a smaller map still stalled) | **Unverified on a real GPU.** Software tests can prove "is this correct?", not "is this fast?" |
| 9.13 | A later look at a screenshot shows old contents | The review tool cached the image by path | Write a new filename each generation |
| 9.14 | A favicon 404 after going live | No favicon | Inline an SVG favicon as a `data:` URL |

### Software-GPU limits (9.15)

- Frame rate is not meaningful: 1–3 FPS will drive dynamic resolution to the floor. Lock the tier when testing pictures.
- First compile is slow: each new material variant can take tens of seconds. Wait enough frames the first time you enter a stage.
- Real performance issues (including 9.12) only show up on a real GPU. **Optional (not documented as used in Pelican World v3):** a cloud browser with a real GPU, or have a person send a shot of the F-key FPS overlay.

### File size (9.16)

See [SIZE.md](../examples/pelican-world/SIZE.md) for the public measurement and the documented unprotected breakdown. Procedural canvases and `DataTexture`s do not enlarge the file (they cost some startup milliseconds). The documented thumbs were JPEG quality 72 at 480×270. Import only the three.js plugins you use.

## Meta-rules for agents

1. Finish one flagship stage to a high standard, then copy sideways. In v2, New York was strongest and Paris weakest; v3 rebuilt Paris to the New York standard.
2. When the user says "cheap" or "not enough detail," turn it into a list: which screen, which element, what is missing.
3. Seed every random source. Reproducibility is what makes before/after shots comparable.
4. Measure the real artifact: bytes, frame counts, note counts, network requests — not a hunch.
5. If you cannot verify it (especially performance under software rendering), say so and hand the check to the user.
