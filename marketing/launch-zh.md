# 宣发草稿（中文）— 尚未发布

**状态：草稿，尚未发布。** 以下文字没有发出、提交或排期。在用户确认原文，并且 `pelican-world/SKILL.md` 已进入默认分支之前，不要发布。仓库地址是 https://github.com/benphant/pelican-world 。

这些是发布草稿，不是已经发出的公告，也不表示已经对着 `main` 成功执行过 `npx skills add`。

## 推荐的首条 X 帖

在同时满足下面三点后再发：（1）`pelican-world/SKILL.md` 已经在 https://github.com/benphant/pelican-world 的默认分支上；（2）准备好官方试玩的短视频或截图真图，而不是示意图；（3）用户确认了下面这段原文。

如果还没有视频或截图，删掉最后一行，不要配假图。

```
我准备公开一个开源的 Agent Skill：Pelican World。

它教编程助手做有电影感、可离线运行的单文件 3D 网页游戏。不依赖 CDN，也没有外部模型、贴图或音频。

参考作品《鹈鹕环游世界》是一个 HTML：6 座程序化城市。公开构建是 1,674,197 字节（十进制 1.67 MB）。手册还记录了未加密版 971,866 字节（约 0.97 MB）；后者没有在本包中复测。

这是一套方法，不是保证每个作品都是 1 MB，或画面等于主机 3A。「挑战用约 1 MB 做出 3A 质感」是努力的方向。

试玩：https://pelican-world.vercel.app/
Skill：https://github.com/benphant/pelican-world
```

若用户希望更短：

```
一个开源 Agent Skill：教 AI 做可离线的单文件 3D 网页游戏。

参考作品《鹈鹕环游世界》：6 座程序化城市，一个 HTML。公开构建 1,674,197 字节。手册中的未加密版约 0.97 MB（十进制）。

试玩：https://pelican-world.vercel.app/
仓库：https://github.com/benphant/pelican-world
```

技能进入默认分支、并且用户确认原文之后再发。在此之前不要把这篇标成已发布。

## 可选的后续 X 帖

草稿 A — 方法（不吹体积）：

```
真正有用的不是某一条“秘技着色器”，而是这个循环：

打出一个 HTML → 在真实浏览器里截图 → 写下哪里看起来廉价 → 只改一处 → 再构建。

截图要等帧计数，不要死等几秒。体积要量最终写到磁盘上的那个文件。
https://github.com/benphant/pelican-world
```

草稿 B — 容易误会的几点（只在出现混淆时再发）：

```
补充三点：

Skill 和教程是 Apache-2.0。游戏本体仍保留全部权利，只提供官方试玩链接。

「挑战用约 1 MB 做出 3A 质感」是方向，不是和商业游戏做基准对比。

网上还有其他「鹈鹕骑自行车」的网页游戏。这一份是 benphant 的《鹈鹕环游世界》：https://pelican-world.vercel.app/
```

## 其他渠道草稿

在用户确认原文，并且 skill 已在默认分支上之前，不要提交。

### GitHub 仓库简介（草稿，尚未写入仓库设置）

```
用 AI Agent 做有电影感、可离线运行的单文件 3D 网页游戏。教程为 Apache-2.0。参考游戏另行授权，试玩：https://pelican-world.vercel.app/
```

建议的 topics（尚未添加）：`agent-skills`、`threejs`、`webgl`、`procedural`、`html5-game`。

### V2EX 标题与正文

标题：

```
开源了一个 Agent Skill：教 AI 做可离线的单文件 3D 网页游戏（附试玩）
```

正文：

```
我整理了一套 Apache-2.0 的 Agent Skill / 教程，让 Cursor、Claude Code、Codex、OpenCode 这类编程助手，按一套可检查的步骤做出「单个 HTML、断网可玩」的 3D 网页游戏：three.js 内联，模型、贴图、音乐都用代码生成，再用无头浏览器截图打磨画面。

参考作品是我的《鹈鹕环游世界》，游戏本体不开放授权，只提供官方试玩：

https://pelican-world.vercel.app/

2026-11-15 测得该公开文件为 1,674,197 字节。手册里还记录了未加密版 971,866 字节（约 0.97 MB，按 1 MB = 1,000,000 字节），这一份我没有对着公开文件复测。我不会保证用这套 skill 做出来的新游戏一定是 1 MB，或达到商业 3A 的画质——那是我在追的方向。

Skill 与教程：https://github.com/benphant/pelican-world

说明：网上还有其他鹈鹕骑车的网页游戏，这一份是 benphant 的《鹈鹕环游世界》。
```

### 即刻 / 小红书（更短，需配真试玩视频或截图）

```
做了一个教 AI 写单文件 3D 游戏的开源 Skill。

参考作《鹈鹕环游世界》：一个 HTML，6 座程序化城市，公开版 1,674,197 字节。游戏本身不开放源码，可以先玩：https://pelican-world.vercel.app/

教程开源地址：https://github.com/benphant/pelican-world
「约 1 MB 的 3A 质感」是我在追的目标，不是保证。
```

小红书尤其依赖视频；没有实机画面就先不要发。

### Bilibili 视频简介（需先有 15–30 秒实机录像）

标题：

```
一个 HTML 的 3D 游戏：Pelican World / 教 AI 做单文件网页游戏的开源 Skill
```

简介：

```
试玩：https://pelican-world.vercel.app/
Skill（Apache-2.0）：https://github.com/benphant/pelican-world

公开构建实测 1,674,197 字节（2026-11-15）。手册记录的未加密版为 971,866 字节。游戏本体保留全部权利，本视频只展示官方试玩；开源的是教程与 Agent Skill。
```

## 分阶段计划

1. **发任何帖之前：** 公开仓库是 https://github.com/benphant/pelican-world 。等到本包进入默认分支、试玩链接仍然可打开、README 把试玩放在安装说明之前、许可表述正确。强烈建议先从官方试玩录一段 15–30 秒实机视频（不要用假截图）。
2. **上线当日只发一条：** 用户确认原文后的推荐首条 X 帖。不要在同一小时里再发 V2EX、Show HN 或 Reddit。
3. **当天稍后：** 有人问安装方式时，用仓库上线后的 `llms.txt` / `npx` 命令回复，不要凭草稿记忆。
4. **第 2–3 天：** 一条讲方法的后续帖（草稿 A），或一条带实机录像的 Bilibili / 短视频。不要多渠道同文群发。
5. **skill 已在默认分支上，并经过一天自检之后：** 再考虑 V2EX、即刻，或英文的 Show HN / r/threejs（见 `launch-en.md`）。一次一个渠道，每次确认原文。
6. **不要**虚构尚未存在的媒体报道、Discord 或社群。**不要**宣称已经上热榜、有安装量，或和其他「鹈鹕骑车」游戏比高下。区分方式写清即可：Pelican World / 《鹈鹕环游世界》，作者 benphant。

## 上线当日用到的链接

- 试玩（2026-11-15 已核实）：https://pelican-world.vercel.app/
- Skill 仓库：https://github.com/benphant/pelican-world
- 体积说明（合并进 `main` 之后的默认分支路径）：https://github.com/benphant/pelican-world/blob/main/pelican-world/examples/pelican-world/SIZE.md
