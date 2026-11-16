# Pelican World

**面向 AI Agent 的技能：做有电影感、可离线运行的单文件 3D 网页游戏。**

挑战用约 1 MB 做出 3A 质感。这句话是追求的方向，不是文件体积或画面质量的保证。

<p align="center">
  <a href="https://pelican-world.vercel.app/"><strong>试玩《鹈鹕环游世界》↗</strong></a>
  &nbsp;·&nbsp;
  <a href="#安装-skill">安装 skill</a>
  &nbsp;·&nbsp;
  <a href="./README.md"><strong>English</strong></a>
</p>

本仓库开源一套 **Agent skill 与教程**：教编程助手做出一个 **单独的 HTML 文件** 就能玩的 3D 网页游戏——不依赖 CDN，也没有外部模型、贴图或音频。参考作品是作者的游戏《鹈鹕环游世界》（Pelican World）。

## 先试玩

游戏本体是另一份作品，仍由 benphant 保留全部权利（© 2026），**不包含**在本 skill 包中。

**[打开官方试玩 ↗](https://pelican-world.vercel.app/)**

按作者的说明，这是一款 3D 跑酷：鹈鹕骑自行车穿过六座程序化城市（纽约 → 长城 → 日本 → 夏威夷 → 埃及 → 巴黎），目标是单 HTML、断网也能玩。

该公开文件在 2026-11-15（Asia/Shanghai）测得的体积：

| 构建 | 字节 | 十进制 MB（1 MB = 1,000,000 字节） | 说明 |
|---|---|---|---|
| 上方链接的公开可玩 HTML | **1,674,197** | 1.674197 MB | 已实测（`Content-Length` 与完整 GET）。对应手册中的加密 / 混淆版。 |
| v3 未加密 HTML | **971,866** | 0.971866 MB（约 0.97 MB） | 来自作者手册的记录；**未**在本包中对照公开文件复测。 |

请注意与二进制 MiB（1 MiB = 1,048,576 字节）的区别：1,674,197 B ≈ 1.60 MiB，971,866 B ≈ 0.93 MiB。更完整的说明，以及本包**没有**验证的内容（可玩流程、画质），见 [`pelican-world/examples/pelican-world/SIZE.md`](pelican-world/examples/pelican-world/SIZE.md)。

本 skill **不保证**新做的游戏一定是 1 MB，也不保证画面能达到商业主机游戏的水准。这些是需要瞄准、再靠测量说话的目标。

## 这个 skill 是什么

一份较短的 [`SKILL.md`](pelican-world/SKILL.md)，加上按需阅读的参考文档：构建与画质分档、光影、动画、材质、世界构建、音频、UI、测试，以及通用的发布说明。

方法可以概括成：每一步结束都在磁盘上留下能玩的 HTML；用无头浏览器截图验收（等的是**帧计数**，不是固定等待）；用物理天空给场景打光，并烘焙成环境贴图；动作从物理量推导，而不是靠关键帧；表面用「三层画法」加噪声；音乐用 Web Audio 合成；在离线浏览器里断言没有外部网络请求。

只要 Agent 能加载本地或 GitHub 上的 skill（Cursor、Claude Code、Codex CLI、OpenCode 等）就可以用。本包不绑定某一个模型。

## 安装 skill

### 本地复制

如果你已经有这个文件夹，把包含 `SKILL.md` 的 `pelican-world/` 目录复制到 Agent 会扫描的 skills 路径。下面的目录与 skills CLI 的 agent 映射一致：

- Cursor：`.agents/skills/pelican-world` 或 `~/.cursor/skills/pelican-world`
- Codex：`.agents/skills/pelican-world` 或 `~/.agents/skills/pelican-world`
- Claude Code：`.claude/skills/pelican-world` 或 `~/.claude/skills/pelican-world`
- OpenCode：`.agents/skills/pelican-world` 或 `~/.config/opencode/skills/pelican-world`

不要覆盖目标位置里已有的、非本 skill 管理的文件。然后对 Agent 说：

```text
使用 Pelican World skill。请阅读 pelican-world/SKILL.md，帮我开始做一个单文件 3D 网页游戏。
```

也可以让 Agent 阅读本地的 [`llms.txt`](llms.txt) 来完成安装。该文件只说明如何安装，并会在复制前请你确认范围。

### GitHub

仓库：[https://github.com/benphant/pelican-world](https://github.com/benphant/pelican-world)

```bash
npx skills add benphant/pelican-world --skill pelican-world
```

这是文档中的 skills CLI 写法：GitHub `owner/repo`，再用 `--skill` 选择 `SKILL.md` 里的 `name`。同一个来源也可以写成完整 GitHub URL。若要指定 Agent 并跳过确认：

```bash
npx -y skills@latest add https://github.com/benphant/pelican-world --skill pelican-world --agent cursor --global --copy --yes
```

把 `cursor` 换成 `codex`、`claude-code` 或 `opencode` 即可。只想装在当前项目里就去掉 `--global`。

CLI 读取的是默认分支。请在 `pelican-world/SKILL.md` 已经位于 `main` 之后再使用远程命令。如果你已经有这个文件夹，而 `main` 上还没有该 skill，请用上面的本地复制。本文没有记录一次已经对着 `main` 完成的安装。

```text
请为我安装 Pelican World skill，先阅读 https://raw.githubusercontent.com/benphant/pelican-world/main/llms.txt
```

## 安装之后

把你的主题、玩法、关卡和署名告诉 Agent。可复用的起始提示词在 [`pelican-world/references/prompt-template.md`](pelican-world/references/prompt-template.md)。

Agent 应当经常重新构建，给你看截图联系表，并报告**磁盘上 HTML 的真实字节数**。在这些数字出现之前，体积和画质都不应当被说成已证实的事实。

## 许可，以及本包不含什么

- **本包中的 skill、教程和文档：** [Apache License 2.0](LICENSE)。见 [NOTICE](NOTICE) 与 [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)。
- **《鹈鹕环游世界》游戏、其源码以及任何构建出的游戏 HTML：** 不包含在内，也不适用 Apache-2.0。© 2026 benphant，保留所有权利。试玩地址：[https://pelican-world.vercel.app/](https://pelican-world.vercel.app/)。
- **第三方参考演示**（包括原手册中称为 `reference.html` 的文件）：不包含，也不得再分发。
- 若你按教程把 [three.js](https://threejs.org/) 打进自己的 HTML，必须保留其 MIT 许可声明。本包不内置 three.js。

Apache 许可并不授权你把「Pelican World」或「鹈鹕环游世界」当作一般商标或标识来使用。

## 目录结构

```
LICENSE  NOTICE  THIRD_PARTY_NOTICES.md  README.md  README_ZH.md
llms.txt  CHANGELOG.md
pelican-world/SKILL.md
pelican-world/references/          # 按需阅读的技术说明
pelican-world/examples/pelican-world/SIZE.md
marketing/                         # 尚未发布的宣发草稿
```

## 参与和宣发

欢迎对 **skill 与教程** 提 issue 和改进。游戏客户端不在贡献范围内，除非作者另行说明。

各渠道宣发草稿在 [`marketing/`](marketing/)。它们**还没有发布**，请不要当成已经发出的公告。
