# capture-windows-screen-skill

An OpenClaw skill for capturing the **Windows host desktop from WSL2**.

中文 | [English](#english)

## 快速下载 / Quick Download

- Source repo: <https://github.com/Hansheng-Li/capture-windows-screen-skill>
- Packaged `.skill`: <https://github.com/Hansheng-Li/capture-windows-screen-skill/releases/download/v0.1.0/capture-windows-screen.skill>

## 中文

### 这是什么

这是一个给 OpenClaw 用的自定义 skill，用来在 **Windows + WSL2** 环境里，从 WSL 侧调用 Windows PowerShell 辅助脚本，抓取当前 **Windows 桌面截图**，再把截图路径返回给 agent 继续分析或发送。

它适合这样的场景：

- OpenClaw 跑在 **Windows 的 WSL2** 里
- agent 能访问 Linux 侧工作区，但你想截的是 **Windows 当前桌面**
- 你想让 agent 帮你“看看你屏幕上现在是什么”
- 你想把截图进一步发回 Web UI、Telegram 或其他聊天渠道

### 适用场景

这个 skill 最适合下面几类请求：

- “截图给我”
- “看看我现在屏幕上有什么”
- “把当前 Windows 桌面截下来”
- “先截图，再帮我分析这个界面”

它尤其适合 **OpenClaw 本体跑在 WSL**，但真正需要截图的是 **Windows 宿主机桌面** 的情况。

### 它是怎么工作的

核心链路很简单：

1. 在 WSL 里执行 `scripts/capture-windows-screen.sh`
2. 这个 shell 脚本调用 Windows PowerShell：
   - `/mnt/c/Windows/System32/WindowsPowerShell/v1.0/powershell.exe`
3. PowerShell 再调用你本机上的辅助脚本：
   - `C:\OpenClaw\capture-screen.ps1`
4. 辅助脚本把截图写到：
   - `C:\OpenClaw\latest-screen.png`
5. WSL 再从：
   - `/mnt/c/OpenClaw/latest-screen.png`
   读取图片，并复制到一个 staging 目录
6. 最后把 staging 后的路径打印出来，给 OpenClaw 后续使用

### 仓库内容

```text
capture-windows-screen-skill/
├── README.md
├── capture-windows-screen/
│   ├── SKILL.md
│   └── scripts/
│       └── capture-windows-screen.sh
└── dist/
    └── capture-windows-screen.skill
```

### 安装方式

最直接的方式是把 `capture-windows-screen/` 目录放到：

```bash
~/.openclaw/workspace/skills/
```

也就是最后变成：

```bash
~/.openclaw/workspace/skills/capture-windows-screen/
```

仓库里也附带了已经打包好的：

```text
dist/capture-windows-screen.skill
```

### 使用方式

#### 1. 只做分析 / 本地预览

```bash
bash scripts/capture-windows-screen.sh
```

默认会把截图复制到：

```text
/home/lhs/.openclaw/workspace/tmp-media/
```

这适合：

- Web UI 里预览
- 本地分析截图内容
- 让 agent 读图、识别界面元素

#### 2. 准备用于聊天渠道发送

```bash
STAGE_DIR=/home/lhs/.openclaw/media/outbound bash scripts/capture-windows-screen.sh
```

这适合：

- Telegram
- 其他依赖 OpenClaw 媒体投递链路的渠道

### 踩过的坑

#### 坑 1：浏览器里能看到，不代表聊天渠道也能收到

一开始把截图放在工作区里的 `tmp-media/`，Web UI 能正常预览，但聊天渠道不一定会把这张图当成“可投递附件”。

**解决办法：**

对“要发到聊天渠道”的截图，优先转存到：

```text
~/.openclaw/media/outbound/
```

也就是让文件进入 OpenClaw 管理的 outbound media store，再走消息投递。

#### 坑 2：本地路径可读，不等于渠道一定会上传成功

在 WSL 里能读到文件，只说明 Linux 侧路径没问题，不代表下游渠道已经把它识别成附件并成功上传。

**解决办法：**

把“截图成功”和“渠道投递成功”当成两件事分开处理：

- 先确认截图文件已经生成
- 再确认它被放到了合适的 staging / outbound 路径
- 最后再让渠道发送

#### 坑 3：PowerShell 路径和辅助脚本路径是环境相关的

这个 skill 假设下面这些路径存在：

- `/mnt/c/Windows/System32/WindowsPowerShell/v1.0/powershell.exe`
- `C:\OpenClaw\capture-screen.ps1`
- `C:\OpenClaw\latest-screen.png`
- `/mnt/c/OpenClaw/latest-screen.png`

如果你的机器路径不同，需要按你的环境修改。

### 适合什么时候用

推荐在下面这些条件同时满足时使用：

- 你用的是 OpenClaw
- OpenClaw 跑在 WSL2 里
- 你需要截取的是 Windows 宿主机桌面
- 你希望把截图继续交给 agent 做分析或回传

如果你只是想在纯 Linux 桌面里截图，或者不是从 WSL 调 Windows，这个 skill 就未必是最合适的方案。

---

## English

### What this is

This is a custom OpenClaw skill for **Windows + WSL2** setups. It lets an agent running inside WSL trigger a Windows PowerShell helper, capture the current **Windows desktop screenshot**, and return a staged image path for analysis or delivery.

It is useful when:

- OpenClaw runs inside **WSL2 on Windows**
- the agent can access the Linux workspace, but the screen you want is the **Windows host desktop**
- you want the agent to inspect what is currently visible on screen
- you want to forward the screenshot to Web UI, Telegram, or another messaging surface

### Good fit scenarios

This skill is a good match for prompts like:

- “Take a screenshot for me”
- “Show me what is on my screen right now”
- “Capture the current Windows desktop”
- “Take a screenshot first, then analyze the UI”

It is especially useful when **OpenClaw runs inside WSL**, while the actual target display is the **Windows host desktop**.

### How it works

The flow is intentionally simple:

1. Run `scripts/capture-windows-screen.sh` inside WSL
2. The shell script calls Windows PowerShell at:
   - `/mnt/c/Windows/System32/WindowsPowerShell/v1.0/powershell.exe`
3. PowerShell runs a host-side helper script at:
   - `C:\OpenClaw\capture-screen.ps1`
4. The helper saves the screenshot to:
   - `C:\OpenClaw\latest-screen.png`
5. WSL reads the mirrored file from:
   - `/mnt/c/OpenClaw/latest-screen.png`
6. The script copies the image into a staging directory and prints the final staged path

### Repository layout

```text
capture-windows-screen-skill/
├── README.md
├── capture-windows-screen/
│   ├── SKILL.md
│   └── scripts/
│       └── capture-windows-screen.sh
└── dist/
    └── capture-windows-screen.skill
```

### Installation

The simplest option is to copy `capture-windows-screen/` into:

```bash
~/.openclaw/workspace/skills/
```

So the final path becomes:

```bash
~/.openclaw/workspace/skills/capture-windows-screen/
```

The repository also includes a packaged artifact at:

```text
dist/capture-windows-screen.skill
```

### Usage

#### 1. Analysis / local preview

```bash
bash scripts/capture-windows-screen.sh
```

By default, the screenshot is copied into:

```text
/home/lhs/.openclaw/workspace/tmp-media/
```

This is a good fit for:

- Web UI preview
- local image inspection
- asking the agent to analyze the screenshot content

#### 2. Delivery-friendly staging for chat channels

```bash
STAGE_DIR=/home/lhs/.openclaw/media/outbound bash scripts/capture-windows-screen.sh
```

This is the safer choice for:

- Telegram
- other channels that rely on OpenClaw's managed outbound media pipeline

### Pitfalls we hit, and how we fixed them

#### Pitfall 1: visible in the browser does not mean deliverable to chat apps

At first, the screenshot was staged only in workspace `tmp-media/`. The Web UI could preview it, but chat channels did not necessarily treat it as a proper outbound attachment.

**Fix:**

For anything that should be delivered through a messaging channel, prefer staging it into:

```text
~/.openclaw/media/outbound/
```

That puts the file inside OpenClaw's managed outbound media store before delivery.

#### Pitfall 2: a readable local file is not the same as a successful channel upload

Just because WSL can read the image file does not mean the downstream channel has recognized it as media and uploaded it successfully.

**Fix:**

Treat these as separate steps:

- confirm the screenshot file exists
- confirm it was restaged into the right staging / outbound directory
- only then send it through the target channel

#### Pitfall 3: the PowerShell and helper paths are environment-specific

This skill assumes the following paths exist:

- `/mnt/c/Windows/System32/WindowsPowerShell/v1.0/powershell.exe`
- `C:\OpenClaw\capture-screen.ps1`
- `C:\OpenClaw\latest-screen.png`
- `/mnt/c/OpenClaw/latest-screen.png`

If your environment differs, update the paths accordingly.

### When to use this skill

Use it when all of the following are true:

- you are using OpenClaw
- OpenClaw runs inside WSL2
- you need a screenshot of the Windows host desktop
- you want the result to be analyzed or returned by the agent

If you only need a screenshot from a pure Linux desktop, or you are not crossing the WSL-to-Windows boundary, this may not be the right skill.
