# Launch drafts (English) — not published

**Status: draft. Not published.** Nothing here has been posted, submitted, or scheduled. Do not post these texts until the user approves the exact wording and `pelican-world/SKILL.md` is on the default branch. The repository URL is https://github.com/benphant/pelican-world.

These are launch drafts, not a live announcement, and not a record that `npx skills add` has been run against `main`.

## Recommended first X post

Use this as the first post only when (1) `pelican-world/SKILL.md` is on the default branch of https://github.com/benphant/pelican-world, (2) a short gameplay video or a still of the official demo is ready to attach, and (3) the user has approved this exact wording.

If there is still no video or still, delete the last line rather than attaching a mockup.

```
I’m publishing an open-source agent skill: Pelican World.

It teaches a coding agent to build cinematic, offline, single-file 3D web games. No CDN, no external models, textures, or audio.

The reference game is one HTML file with 6 procedural cities. The public build is 1,674,197 bytes (1.67 decimal MB). The notes also record an unprotected build of 971,866 bytes (~0.97 MB); that file has not been re-measured here.

This is a method, not a promise that every game will be 1 MB or look like a console title. I’m chasing AAA-style visuals in ~1 MB.

Play: https://pelican-world.vercel.app/
Skill: https://github.com/benphant/pelican-world
```

Character count is short enough for a single X post with a link preview. Prefer attaching a real clip of the official demo over a second link.

If the user wants a shorter alternative:

```
An open-source agent skill for cinematic, offline, single-file 3D web games.

Reference game, Pelican World: 6 procedural cities, one HTML file. Public build 1,674,197 bytes. Documented unprotected build ~0.97 decimal MB.

Play: https://pelican-world.vercel.app/
Skill: https://github.com/benphant/pelican-world
```

Drop any “soon” wording on launch day, after the skill is on the default branch. Do not post this draft until the user approves the text.

## Follow-up X posts (optional, later)

Draft A — how the method works (no size boast):

```
The useful part isn’t a secret shader. It’s the loop:

build one HTML file → screenshot it in a real browser → write down what looks cheap → change one thing → rebuild.

Wait on frame counts, not a fixed timeout. Measure the file you actually wrote.
https://github.com/benphant/pelican-world
```

Draft B — what the skill is not:

```
A few clarifications:

The skill and tutorial are Apache-2.0. The game itself stays all-rights-reserved; I only link the official playable build.

“Chasing AAA-style visuals in ~1 MB” is a direction. I am not claiming a benchmark against commercial games.

There are other pelican-on-a-bike web games. This one is Pelican World, by benphant: https://pelican-world.vercel.app/
```

Do not post Draft B unless confusion actually shows up.

## Other channel drafts

Do not submit any of these until the user approves the exact text and the skill is on the default branch.

### GitHub (repository description draft — not applied)

```
Agent skill for cinematic, offline, single-file 3D web games. Tutorial under Apache-2.0. Play the reference game (separately licensed): https://pelican-world.vercel.app/
```

Suggested topics (not applied): `agent-skills`, `threejs`, `webgl`, `procedural`, `html5-game`.

### skills.sh / `npx skills add` blurb

```
Pelican World — an agent skill for building playable, offline, single-file 3D web games with procedural assets and a screenshot review loop. The reference game is separately licensed.
```

### Show HN (title + first paragraph)

Title:

```
Show HN: Pelican World – an agent skill for single-file, offline 3D web games
```

Body:

```
I wrote an agent skill (Apache-2.0) that walks a coding agent through making a playable 3D game as one HTML file: three.js inlined, procedural models/textures/audio, a physical sky, and a headless screenshot loop.

The reference game, Pelican World, is separately licensed (all rights reserved). You can play the official build here:

https://pelican-world.vercel.app/

I measured that public file at 1,674,197 bytes on 15 Nov 2026. The notes also describe an unprotected build of 971,866 bytes (~0.97 decimal MB), which I have not re-measured against a public file. I’m not claiming every game built with the skill will hit those sizes, or that the visuals match a commercial “AAA” title — that’s the direction I’m chasing.

Skill and tutorial: https://github.com/benphant/pelican-world
```

### Reddit r/threejs or r/proceduralgeneration (title + body)

Title:

```
[Skill] Pelican World: a method for offline, single-file 3D games in three.js (playable demo linked)
```

Body: reuse the Show HN paragraph, add that the skill is for Cursor / Claude Code / Codex / OpenCode, and state clearly that the game HTML is not in the repo.

### Hacker News / Twitter-adjacent communities

Do not cross-post the same paragraph to five places on the same day. See the staged plan below.

## Staged plan (practical)

1. **Before any post:** the public repository is https://github.com/benphant/pelican-world. Wait until this package is on the default branch, the play URL still works, the README leads with the demo, and the license lines are correct. Optional but strongly recommended: a 15–30 s gameplay clip captured from the official demo (no fake screenshots).
2. **Launch day, post 1 only:** the recommended first X post, after the user approves that exact text. Do not submit Show HN or Reddit in the same hour.
3. **Same day, later:** if people ask how to install, reply with the live `llms.txt` / `npx` command from the published repo, not from memory of this draft.
4. **Day 2–3:** one longer post (Draft A) or a short video on a video-first channel (YouTube / Bilibili — Chinese drafts are in `launch-zh.md`). Still no spam.
5. **Only after the skill is on the default branch and has had a day of sanity checking:** optional Show HN, r/threejs, or a Chinese forum post (see `launch-zh.md`). One channel at a time, exact text approved each time.
6. **Do not** invent press, Discord, or community groups that do not exist. **Do not** claim GitHub Trending, install counts, or comparisons to other “pelican bike” games. Name and author are enough to tell them apart: Pelican World, by benphant.

## Link list for launch day

- Playable game (live, verified 2026-11-15): https://pelican-world.vercel.app/
- Skill repo: https://github.com/benphant/pelican-world
- Size notes (on `main` after this package is merged): https://github.com/benphant/pelican-world/blob/main/pelican-world/examples/pelican-world/SIZE.md
