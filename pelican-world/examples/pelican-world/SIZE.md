# Pelican World build sizes

This file records byte counts for the author's reference game, Pelican
World v3. It is a measurement and documentation log, not a promise that
games built with this skill will be the same size.

All megabyte figures below use **decimal MB** (1 MB = 1,000,000 bytes),
not binary mebibytes (1 MiB = 1,048,576 bytes). Both are listed so the
two are not confused.

## Public playable build (measured)

| Field | Value |
|---|---|
| URL | https://pelican-world.vercel.app/ |
| Date of measurement | 2026-11-15 (Asia/Shanghai) |
| HTTP status | 200 |
| `Content-Type` | `text/html; charset=utf-8` |
| Bytes | **1,674,197** |
| Decimal MB | 1.674197 MB |
| Binary MiB | 1,674,197 ÷ 1,048,576 ≈ 1.5966 MiB |
| How measured | HTTP `Content-Length` and a full GET of the document, both 1,674,197 bytes |
| External script `src` | none observed in that document |

This total matches the **protected / obfuscated** size written in the
author's methodology notes (1,674,197 B). Response headers on that date
also included `X-Frame-Options: DENY`, `Content-Security-Policy:
frame-ancestors 'none'`, and `X-Robots-Tag: noarchive`. The HTML
comment in that document states that the game is © 2026 benphant, all
rights reserved. The game is **not** included in this skill package.

This measurement does not by itself verify gameplay, visual quality, or
offline behaviour. It verifies the public document's size and that it
was served as a single HTML file on the date above.

A HEAD request on 2026-11-16 still returned HTTP 200 and `Content-Length: 1674197`. That check did not download or re-hash the body, and it did not re-measure the unprotected figure below.

## Unprotected build (documented, not independently verified here)

The methodology notes record an unprotected (unobfuscated) v3 HTML of
**971,866 bytes**.

| Field | Value |
|---|---|
| Bytes (as documented) | **971,866** |
| Decimal MB | 0.971866 MB (often rounded in prose to ~0.97 MB) |
| Binary MiB | 971,866 ÷ 1,048,576 ≈ 0.9267 MiB |
| Status in this package | Documented in the author's notes; **not** re-measured against a public file in this package |
| Public URL | none supplied for the unprotected file |

Documented composition of that unprotected file, also **not**
re-measured here:

| Part | Documented share |
|---|---|
| three.js | about 60% of the file |
| Game code, minified | about 232 KB |
| Six level-select thumbnails | 80 KB as JPEG; about 107 KB after base64 |
| HTML / CSS | about 34 KB |

The same notes record that obfuscation expanded the game code from
about 232 KB to about 745 KB in the protected build, which accounts
for most of the jump from 971,866 B to 1,674,197 B. That expansion and
the unprotected file itself have not been independently re-measured
in this package.

## What this does not claim

- It does not guarantee that a new game built with this skill will be
  1 MB, 0.97 MB, or 1.67 MB.
- It does not compare Pelican World to other games as an "AAA" or
  file-size benchmark.
- Thumbnail, three.js, and game-code breakdowns are from the notes,
  not from a fresh split of the public file.
