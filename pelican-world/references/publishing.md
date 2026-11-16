# Publishing

This page is **generic guidance** for shipping a game you built with the method. It does not document, and you must not invent, project-specific host locks, domain regular expressions, fingerprint hashes, watermark constants, or obfuscator seeds.

A static front-end game that runs in the browser can be saved or copied. There is no absolute technical protection. Be honest about that with the user.

The Pelican World **game** is not part of this skill and remains all-rights-reserved. If the user wants people to play the author's game, link https://pelican-world.vercel.app/ — do not vendor that file into a repository you are about to open-source unless they have explicitly relicensed it.

## Attribution (do this)

Put the author's name (and a year, and a license or reserved-rights line they chose) in all of:

- `<title>` and `<meta name="author">`
- An end card or credits screen
- A console banner
- A `/*! ... */` legal comment that survives minification (`legalComments: 'inline'`)

Keep third-party notices in the bundle, especially the three.js MIT license. See [THIRD_PARTY_NOTICES.md](../../THIRD_PARTY_NOTICES.md).

## Two artifacts, if the user wants a public and a private copy

If the user wants a readable private build and a separate public deploy, treat the public file as a **derived artifact**:

- Build from a copy of the sources, not by overwriting the private HTML.
- Measure and record the byte length of **each** file with `Buffer.byteLength`.
- Do not check a private, readable build into a public repository by accident.

Optional public-build techniques (all imperfect, all optional):

- A **host allowlist** so a copy opened from an unexpected origin can show a credits screen and a link to the official URL. If you add an official domain later, you must rebuild and redeploy. Do not hard-code another project's hosts or preview-domain patterns.
- **Attribution** in meta tags, credits, and legal comments (preferred over trying to hide the program).
- **Obfuscation** of *game* code only, if the user asks. Leave vendor libraries (three.js) minified but not obfuscated: obfuscating them inflates size and can hurt frame rate. If you try this, measure FPS against the plain build and keep the tool's license. Do not publish generator seeds or "self-defending" recipes as if they were required.
- Do **not** use an infinite `debugger` trap. It punishes genuine players and is easy to bypass.

## HTTP headers on a static host

Sensible defaults for a game that should not be wrapped in someone else's iframe:

```json
{
  "headers": [{
    "source": "/(.*)",
    "headers": [
      { "key": "X-Frame-Options", "value": "DENY" },
      { "key": "Content-Security-Policy", "value": "frame-ancestors 'none'" },
      { "key": "X-Content-Type-Options", "value": "nosniff" },
      { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" }
    ]
  }]
}
```

Adapt the file shape to the host (for example `vercel.json`, Netlify `_headers`, or an nginx snippet). An optional `X-Robots-Tag: noarchive` asks search engines not to store a snapshot; use it only if the user wants that.

Add an **inline SVG favicon** (`<link rel="icon" href="data:image/svg+xml,...">`). A missing favicon shows up as a 404 on many hosts.

## Going live

- Deploy the directory that contains the single HTML (and any host config) with whatever tool the user already uses. This skill does not require a particular vendor.
- After deploy, smoke-test the **production URL**: no console page errors, no unexpected external requests, you can enter a stage.
- If the official URL must work in mainland China, `*.vercel.app` and many other foreign hosts are often unreachable there. A custom domain, and for long-term stable access a mainland-licensed server or CDN, is an operational choice for the user — not something this skill can complete on its own.

## What "done" means for a release

See the shipping list in [Checklists](checklists.md). At minimum: the playable file is one HTML document, its byte length is recorded, credit is present, offline and error assertions passed on a production URL, and any claim you make about size or quality has a measurement behind it.
