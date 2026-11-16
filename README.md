# Pelican World

**Cinematic, single-file, offline 3D games — an agent skill.**

Chasing AAA-style visuals in ~1 MB. That line is a direction, not a guarantee of file size or image quality.

<p align="center">
  <a href="https://pelican-world.vercel.app/"><strong>Play Pelican World ↗</strong></a>
  &nbsp;·&nbsp;
  <a href="#install-the-skill">Install the skill</a>
  &nbsp;·&nbsp;
  <a href="./README_ZH.md"><strong>简体中文</strong></a>
</p>

This repository holds an **open-source agent skill and tutorial** that teaches a coding agent how to build a playable 3D web game as **one HTML file**: no CDN, no external models, textures, or audio. The worked example is the author's game *Pelican World* (鹈鹕环游世界).

## Play first

The game itself is a separate work. It remains © 2026 benphant, all rights reserved, and is **not** distributed in this skill package.

**[Play the official build ↗](https://pelican-world.vercel.app/)**

As documented by the author, the game is a 3D runner: a pelican on a bicycle through six procedural cities (New York → the Great Wall → Japan → Hawaii → Egypt → Paris), designed to run offline from a single HTML file.

Size of that public file, measured on 2026-11-15 (Asia/Shanghai):

| Build | Bytes | Decimal MB (1 MB = 1,000,000 bytes) | Notes |
|---|---|---|---|
| Public playable HTML at the link above | **1,674,197** | 1.674197 MB | Measured (`Content-Length` and a full GET). This is the protected / obfuscated build. |
| Unprotected v3 HTML | **971,866** | 0.971866 MB (~0.97 MB) | Documented in the author's notes; **not** independently re-measured against a public file in this package. |

Binary mebibytes (1 MiB = 1,048,576 bytes) are different: 1,674,197 B ≈ 1.60 MiB, 971,866 B ≈ 0.93 MiB. Details and what was *not* verified (gameplay, visual quality) are in [`pelican-world/examples/pelican-world/SIZE.md`](pelican-world/examples/pelican-world/SIZE.md).

This skill does **not** promise that a new game will be 1 MB, or that it will look like a commercial console title. Those are goals to aim at and then measure.

## What the skill is

A short [`SKILL.md`](pelican-world/SKILL.md) plus reference files an agent reads only when needed: build and quality tiers, lighting, animation, materials, world building, audio, UI, testing, and generic publishing.

The method in one paragraph: keep a playable HTML file on disk after every step; review it with headless screenshots (wait on **frame counts**, not fixed timeouts); light the scene with a physical sky that also becomes the IBL; derive motion from physics, not from keyframes; paint surfaces in three layers with noise; synthesize music in the Web Audio API; assert zero external network requests in an offline browser.

Works with agents that can load a local or GitHub-hosted skill (Cursor, Claude Code, Codex CLI, OpenCode, and similar). This package does not pin you to one model.

## Install the skill

### Local copy

If you already have this folder, copy `pelican-world/` (the directory that contains `SKILL.md`) into a skills path your agent scans. These directories match the skills CLI agent map:

- Cursor: `.agents/skills/pelican-world` or `~/.cursor/skills/pelican-world`
- Codex: `.agents/skills/pelican-world` or `~/.agents/skills/pelican-world`
- Claude Code: `.claude/skills/pelican-world` or `~/.claude/skills/pelican-world`
- OpenCode: `.agents/skills/pelican-world` or `~/.config/opencode/skills/pelican-world`

Do not overwrite unmanaged files in the destination. Then ask your agent:

```text
Use the Pelican World skill. Read pelican-world/SKILL.md and help me start a single-file 3D web game.
```

You can also ask the agent to install from this tree by reading the local [`llms.txt`](llms.txt). That file only describes installation. It will ask you to confirm scope before it copies anything.

### GitHub

Repository: [https://github.com/benphant/pelican-world](https://github.com/benphant/pelican-world)

```bash
npx skills add benphant/pelican-world --skill pelican-world
```

That is the documented skills CLI form: GitHub `owner/repo`, then `--skill` with the `name` from `SKILL.md`. The same source accepts a full GitHub URL. To choose an agent and skip prompts:

```bash
npx -y skills@latest add https://github.com/benphant/pelican-world --skill pelican-world --agent cursor --global --copy --yes
```

Replace `cursor` with `codex`, `claude-code`, or `opencode` as needed. Omit `--global` for a project-only install.

The CLI reads the default branch. Use the remote command once `pelican-world/SKILL.md` is on `main`. If you already have this folder and `main` does not yet contain the skill, use the local copy. This README does not record a completed install from `main`.

```text
Install the Pelican World skill for me by reading https://raw.githubusercontent.com/benphant/pelican-world/main/llms.txt
```

## After install

Point the agent at your idea (theme, playable loop, stages, how you want to be credited). A reusable starting prompt is in [`pelican-world/references/prompt-template.md`](pelican-world/references/prompt-template.md).

The agent should rebuild often, show you contact sheets, and report the **byte length of the HTML on disk**. Treat size and quality claims as unverified until those measurements exist.

## License and what is not in this package

- **Skill, tutorial, and documentation in this package:** [Apache License 2.0](LICENSE). See [NOTICE](NOTICE) and [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
- **The Pelican World game, its source, and any built HTML of the game:** not included, not Apache-2.0. © 2026 benphant. All rights reserved. Play it at [https://pelican-world.vercel.app/](https://pelican-world.vercel.app/).
- **Third-party reference demos** (including a file the original notes call `reference.html`): not included and not for redistribution.
- If you follow the tutorial and bundle [three.js](https://threejs.org/), you must keep its MIT license notice in your own HTML. This package does not vendor three.js.

The Apache license does not grant general permission to use the names or logos "Pelican World" or "鹈鹕环游世界".

## Repository layout

```
LICENSE  NOTICE  THIRD_PARTY_NOTICES.md  README.md  README_ZH.md
llms.txt  CHANGELOG.md
pelican-world/SKILL.md
pelican-world/references/          # progressive technical guides
pelican-world/examples/pelican-world/SIZE.md
marketing/                         # unpublished launch drafts
```

## Contributing and launch

Issues and improvements to the **skill and tutorial** are welcome. The game client is out of scope unless the author later says otherwise.

Launch drafts (X and other channels) live in [`marketing/`](marketing/). They are **not published**. Do not treat them as a live announcement.
