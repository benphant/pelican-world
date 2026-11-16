# Checklists

## Texture and lighting (every screenshot)

- [ ] No large flat colors (add layer-2 noise).
- [ ] Wall bases, eaves, and sills are darker (layer-3 contact or AO).
- [ ] Highlights are not identical everywhere (roughness variation).
- [ ] No obvious tiling or regular grid (dual-scale sampling; no regular `sin` terms in procedural textures).
- [ ] Emitters are not blown-out rectangles (emissive × albedo, lower intensity, only a fraction of windows lit).
- [ ] Close-ups are not too smooth (micro-normals, triplanar detail, or `canvasNormal`).
- [ ] Fog is thin enough that shafts and the sky still read.
- [ ] Sun direction serves the shot (back light or side light).
- [ ] Night scenes have fill and ground pools, and the subject is readable.
- [ ] Floating props have contact darkening at the base.

## Runtime

- [ ] `offline: true`: zero external requests, zero `pageerror`.
- [ ] Auto-play (or a careful manual run) can finish every stage, in the intended order.
- [ ] All four quality tiers can be selected; the no-WebGL2 path still runs.
- [ ] Music note counters keep rising after retry, title, fail, clear, mute, and stage changes.
- [ ] Missing `localStorage` does not crash.
- [ ] Desktop and phone UI screenshots: no overflow, no overlap.
- [ ] `prefers-reduced-motion` and safe-area insets are honored.

## Delivery

- [ ] One HTML file; **on-disk byte length recorded**.
- [ ] Credit appears in `<title>`, meta, end card, console, and a `/*! */` legal comment.
- [ ] three.js MIT notice is still in the minified bundle.
- [ ] Production smoke test passed (0 page errors, 0 unexpected external requests).
- [ ] If the audience is in mainland China, the user has an access plan (custom domain or a licensed host). This skill cannot complete ICP filing.

## Documented parameter sheet (Pelican World v3 notes)

These are starting points from the example game, not targets for every new game.

| Area | Parameter | Documented value |
|---|---|---|
| Camera | Near / far / default FOV | 0.15 / 3200 / `58 + speed * 0.55` |
| Camera | Chase distance / height | `5.6 + speed * 0.05` / `2.5 + y * 0.4` |
| Damping λ | Steer / lean / cam pos / cam look / FOV | 9 / 8 / 7 / 12 / 3 |
| Physics | Cruise / sprint / brake | 15 / 24 / 3 (× difficulty `1 + stageIndex * 0.05`) |
| Physics | Jump impulse, gravity, ramp launch | 7.2 / 20 / `9.5 + speed * 0.12` |
| Rider | Wheel radius, gear ratio, thigh / shin | 0.36 / 0.42 / 0.42 / 0.44 |
| Rider | Pouch spring k / c / land impulse | 160 / 6 / −3.5 |
| Sky | Rayleigh / Mie / scale heights | (5.8, 13.5, 33.1)×10⁻⁶ / 21×10⁻⁶ / 8 km, 1.2 km |
| Shadows | bias / normalBias / radius | −0.0003 / 0.035 / 2.2 |
| Fog | Density / falloff | 0.00145–0.0034 / 0.009–0.012 |
| Post | bloom / dirt / flare / ca / vignette / grain / sharpen | 0.07 / 0.12 / 0.14 / 0.0013 / 0.3 / 0.022 / 0.28 |
| Exposure | key / minE / maxE / tau | 0.16 / 0.35 / 3 / 1.3 s |
| Grade | agx / sat / contrast | 0.25 / 1.05 / 0.18 |
| Dyn. res. | Down / up / floor | >26 ms / <17.5 ms / 0.6 |
| Audio | Compressor / music / SFX / reverb | −16 dB 4:1 / 0.42 / 0.75 / 0.32 |
