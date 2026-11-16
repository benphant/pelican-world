# UI

A usable UI language from the documented project (derived from studying a reference demo that is **not** redistributed here): **dark ink backgrounds, bone-white type, one accent color; hairline dividers; letter-spaced uppercase labels; tabular numbers; a darkened scrim behind type so it stays readable on any 3D frame; frosted panels; the same three-layer material idea as the world.**

## Design tokens

```css
:root {
  --ink: #0b0d11;
  --ink2: #141820;
  --bone: #f3eee4;
  --dim: rgba(243,238,228,.66);
  --faint: rgba(243,238,228,.28);
  --hair: rgba(243,238,228,.14);
  --acc: #ffc24a;
  --acc2: #ff8a3d;
  --good: #7ee0a1;
  --bad: #ff5a5f;
  /* --acc changes per stage: gold, red, pink, blue, … */
  --f-ui: "PingFang SC","HarmonyOS Sans SC","MiSans","Microsoft YaHei UI", system-ui, sans-serif;
  /* System fonts only: offline, and usable in mainland China */
  --f-num: "SF Mono","JetBrains Mono","Cascadia Mono","Roboto Mono", ui-monospace, Menlo, Consolas, monospace;
  --r-s: 10px; --r-m: 16px; --r-l: 22px;
  --sp1: 4px; --sp2: 8px; --sp3: 12px; --sp4: 16px; --sp5: 24px; --sp6: 32px; --sp7: 48px;
  --ease: cubic-bezier(.2,.8,.2,1);
  --ease-o: cubic-bezier(.16,1,.3,1);
  --grain: url("data:image/svg+xml;utf8,<svg …><filter id='n'><feTurbulence type='fractalNoise' baseFrequency='.9' numOctaves='3' stitchTiles='stitch'/>…");
}
```

- Numbers use `.num { font-family: var(--f-num); font-variant-numeric: tabular-nums; }` so a ticking score does not shift sideways.
- Eyebrow labels: `font: 600 11px var(--f-lat); letter-spacing: .28em; text-transform: uppercase;`.
- Each stage writes `--acc` from JavaScript so buttons, bars, stars, and the route map change color together.

## Three-layer glass (the paint method, applied to UI)

```css
.glass {
  /* Layer 1: base */
  background:
    radial-gradient(120% 90% at 0% 0%, rgba(255,255,255,.085), transparent 55%),
    linear-gradient(165deg, rgba(30,34,44,.66), rgba(10,12,17,.58));
  backdrop-filter: blur(18px) saturate(1.45) brightness(.92);
  box-shadow:
    0 1px 0 rgba(255,255,255,.06) inset,
    0 -10px 30px -18px rgba(0,0,0,.6) inset,
    0 28px 60px -24px rgba(0,0,0,.7),
    0 2px 6px rgba(0,0,0,.25);
}
.glass::before {
  background-image: var(--grain); opacity: .13; mix-blend-mode: overlay; /* Layer 2: grain */
}
.glass::after { /* Layer 3: 1px lit stroke */
  padding: 1px;
  background: linear-gradient(170deg, rgba(255,255,255,.34), rgba(255,255,255,.06) 32%,
              rgba(255,255,255,.02) 60%, rgba(255,255,255,.12));
  mask: linear-gradient(#000 0 0) content-box exclude, linear-gradient(#000 0 0);
}
```

`backdrop-filter` is very expensive under software rendering; raise screenshot timeouts (the notes used 120 s). Real GPU players are much less affected.

Primary buttons use the same idea:

- A three-stop vertical gradient (lighter top, darker base) derived from the accent with `color-mix(in oklab, ...)`.
- A 1 px inner highlight at the top, a 2 px inner shadow at the bottom.
- A colored outer glow from the accent.
- `::before` soft-light grain.
- Hover: rise 2 px, icon shifts 3 px right; active: sink and scale to 0.985.

## Screen flow (documented example)

```
Loading (wordmark + a 2 px sheen on the bar)
 └→ Title (scr-title): live 3D orbit behind a left-side scrim
     ├ primary "start" / control legend (<kbd> caps) / quality (auto, high, …)
     ├ route map + total stars + high score
     └ stage select: cards with inlined thumbs, index, names, stars, best score
       (hover: lift, glow, image scale 1.07)
 └→ Stage card (scr-card): map highlight, time-of-day, a short fact, stats,
    "go" disabled until the stage has loaded
 └→ 3-2-1 countdown (pop + a cue) → in-game HUD
 └→ Finish (orbit camera 4.2 s) → single-stage results, or the next card in tour
                                  → last stage: fireworks → victory (total stars, map)
 Pause (continue / retry / title / quality)    Fail (score, back to title)
```

- Transitions: `scrIn` fade, children `.rise.d1…d4` staggered 60 ms.
- Stage records in `localStorage` (the example key was `pelican_stages`: best score and stars per stage). Wrap every `localStorage` access in `try/catch` so `file://` and private mode do not crash.
- Thumbs in the example were 480×270 JPEG quality 72, inlined at build time as `window.__THUMBS`. Keep them small; they become about 107 KB after base64 in the documented unprotected file.

## HUD

- Top left: stage progress (`prog-mark` for position), "stage N".
- Top right: score (tabular figures, a `bump` scale to 1.14 in the accent on gain), combo, speed bar, mute, pause.
- Center: toast (`up` float-and-fade) and big type (`pop`, tracking from 0.3 em to 0.08 em).
- Touch: left, right, brake, jump (`pointerdown` plus `preventDefault`), shown only on `pointer: coarse`.
- F shows FPS, tier, and dynamic-resolution percent. C cycles cameras, P/Esc pauses, M mutes.

## Responsive

Breakpoints in the example: width 1180, 1000, 760 px; height 860, 560 px.

- Stage cards: 6 columns, 3 at ≤1180 px, a horizontal `scroll-snap` row at ≤760 px. At ≤1000 px hide the side map and stack the card screen.
- Short screens (≤860 px) hide the lead paragraph; ≤560 px shrinks the title and hides the map.
- Honor `prefers-reduced-motion`.
- Use `env(safe-area-inset-*)` for notched phones.

A UI screenshot script (`uishot.mjs` in the notes) defaulted to 1440×810; a `mobile` flag enabled `hasTouch` / `isMobile` and captured title, cards, HUD, pause, and results at a phone size, checking overflow and overlap.

## Route map (optional, Canvas 2D)

The documented map: a dark ocean with a lat/long grid; land as a Path2D with a drop shadow; stipple clipped to land; cities projected from real lat/long; a dashed route; the current leg a glowing accent (`shadowBlur 12`) drawn on progressively.
