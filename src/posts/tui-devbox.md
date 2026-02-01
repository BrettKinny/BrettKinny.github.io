---
title: "Building a TUI Devbox: My Claude-Powered Terminal Development Environment"
description: "How I built a portable, reproducible Docker-based terminal development environment packed with modern CLI and TUI tools, designed for Claude Code YOLO mode from an iPad"
date: 2026-01-28
layout: post.njk
---

*Or how Claude made me stop worrying and love the terminal.*

![Claude Code running on an iPad](/images/claude-ipad.png)

It's 2026 and there's a new UI on the block. Forget React, Angular, Flutter, React Native, MAUI, Electron, SwiftUI... the next bandwagon to jump on is the oldest of them all—the terminal.

I for one am 100% onboard with this trend, and am systematically working towards replacing every piece of software in my life with a TUI. I've been trying to improve my keyboard-only chops slowly for years, and I hate many modern web apps and UIs. Just give me the terminal. Strap me in, and plug me straight into the matrix.

As a Windows kid, the terminal is not my native environment. Sure, I knew how to use DOS to run Wolf3D, but since then I've grown accustomed to the GUI. That's changing now—fast.

This mini project started because I wanted to be able to run [Claude Code](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview) in YOLO mode from my iPad. The iPad doesn't have a proper shell, so I need to SSH into a server. What I set out to do was create a "TUI devbox"—a Docker container I can connect to from my iPad (or anywhere) that contains all the tools I need to develop software in a terminal environment.

This article is for:

- People that want to YOLO with Claude in an isolated environment
- People that want to YOLO with Claude on their iPad
- People that are interested in the current crop of CLI/TUI tools
- People that want to dabble in the terminal, just get their toes wet, without having to learn VIM
- Vibe coders that need a safe sandbox (please run this in a container, not on your local machine)

I'm not going to hold your hand too much—you can ask your favourite LLM about Docker or any acronyms you don't know. That's kind of the point.

---

## Why a Container?

Three reasons:

**Reproducible environment.** I can spin this up anywhere—my home [Unraid](https://unraid.net/) server, my [Hostinger](https://www.hostinger.com/) VPS that I picked up cheap during the Black Friday sales, a mate's machine. Same tools, same config, every time.

**Keeps the host clean.** The container holds the tools. Your projects live outside the container. Blow away the container whenever you want—your work survives. This is the key mental model: the devbox is disposable, the code is not.

**AI loves it.** Here's something that took me a while to properly appreciate: Claude, Gemini, Codex—they can read terminal output like it's their first spoken language. Long error logs, dependency conflicts, build failures—just paste the output and the AI will parse it and suggest the next step. An isolated container means you can let the AI go wild without worrying about it hosing your system.

---

## The Tools

If I'm going to make a TUI devbox, I wanted the best CLI and TUI tools I could find as of January 2026. All the bells and whistles, but not crazy bloated. Modern luxuries like mouse support—this is not one for the VIM heads.

### Base Image

I started with `ubuntu:24.04`. I could probably optimise this, but I wanted something I know is going to work. No point over-engineering the foundation.

### CLI Tools

| Tool | Language | What It Replaces | GitHub |
|------|----------|-----------------|--------|
| [fd](https://github.com/sharkdp/fd) | Rust | `find` | Fast, user-friendly alternative |
| [ripgrep (rg)](https://github.com/BurntSushi/ripgrep) | Rust | `grep` | Blazingly fast recursive search |
| [bat](https://github.com/sharkdp/bat) | Rust | `cat` | Syntax highlighting and git integration |
| [zoxide](https://github.com/ajeetdsouza/zoxide) | Rust | `cd` | Learns your habits, jump anywhere |
| [eza](https://github.com/eza-community/eza) | Rust | `ls` | Icons, colours, git status |
| [delta](https://github.com/dandavison/delta) | Rust | `diff` | Syntax-highlighting pager for git diffs |
| [xh](https://github.com/ducaale/xh) | Rust | `curl` | Friendly HTTP client |
| [gh](https://github.com/cli/cli) | Go | — | GitHub's official CLI |
| [yq](https://github.com/mikefarah/yq) | Go | — | YAML/JSON/XML processor |
| [oh-my-posh](https://github.com/JanDeDobbeleer/oh-my-posh) | Go | boring prompt | Cross-shell prompt theme engine |
| [jq](https://github.com/jqlang/jq) | C | — | Lightweight JSON processor |

You'll notice the Rust renaissance here. Basically every classic Unix tool is getting a Rust rewrite that's faster, more user-friendly, and has sane defaults. The Go tools fill in the gaps where Rust hasn't gotten to yet.

### TUI Tools

| Tool | Language | Description | GitHub |
|------|----------|-------------|--------|
| [zellij](https://github.com/zellij-org/zellij) | Rust | Terminal multiplexer (tmux alternative) |
| [lazygit](https://github.com/jesseduffield/lazygit) | Go | Terminal UI for git |
| [superfile (spf)](https://github.com/yorukot/superfile) | Go | Terminal file manager |
| [micro](https://github.com/zyedidia/micro) | Go | Terminal text editor |
| [fzf](https://github.com/junegunn/fzf) | Go | Interactive fuzzy finder |
| [htop](https://github.com/htop-dev/htop) | C | Interactive process viewer |

---

## My Thinking, Step by Step

Let me walk through the logic behind the build, because I think the *why* is more useful than just handing you a Dockerfile.

### Give Claude Access to Modern CLI Power

The whole point of this devbox is to pair Claude Code with the best terminal tools available. When Claude has access to `ripgrep` instead of `grep`, `fd` instead of `find`, and `bat` instead of `cat`, it writes better commands. It gets syntax-highlighted output. It can parse structured data with `jq` and `yq`. You're giving it a sharper set of knives.

### Separate the Container from the Code

This is the most important architectural decision. Your projects live on the host (or in persistent storage), mounted into the container:

```bash
docker run -it --name devbox \
  -v ~/workspace:/workspace \
  devbox /bin/bash
```

The container is the workshop. The `/workspace` mount is where the actual work lives. You can nuke and rebuild the container any time—new tools, different versions, start fresh—and your project files don't care.

### BYO SDK

The container is intentionally minimal on language runtimes. Mount your project folder and install what you need for that specific project. Working on a Node project? Install Node. .NET project? Install the SDK. The devbox gives you the *environment*, not the *stack*.

### The CLAUDE.md File

[Claude Code](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview) reads a `CLAUDE.md` file for project context. I put one at `~/.claude/CLAUDE.md` with a list of all the CLI tools available in the container—their names, what they do, and the aliases I've set up. This means Claude knows it can use `rg` instead of `grep`, `fd` instead of `find`, and so on. Small thing, big difference.

```markdown
# Devbox Environment

## Available CLI Tools
- fd - modern find
- rg - ripgrep, fast search
- bat - cat with syntax highlighting
- eza - modern ls (aliased to ls)
- lazygit - Git TUI
- spf - Superfile file manager
- micro - terminal text editor
- delta - better git diffs

## Aliases
- ls → eza --icons
- ll → eza -la --icons
- cat → bat --paging=never
- claude-yolo → claude --dangerously-skip-permissions
- lg → lazygit
```

---

## The Tricky Bits

### Running Claude in YOLO Mode

Claude Code has a `--dangerously-skip-permissions` flag that lets it run commands without asking for confirmation. Essential for flow state. But it only works when running as a non-root user. The Dockerfile creates a `dev` user specifically for this:

```bash
alias claude-yolo='claude --dangerously-skip-permissions'
```

YOLO mode on an iPad. What a time to be alive.

**Disclaimer:** I don't know if this environment will enhance Claude Code's performance (speed-wise, probably negligible) or harm it (because these tools might not be in the training data). It works great for me either way.

### Authentication Lives Outside

Your `.gitconfig`, SSH keys, and GitHub auth tokens should live on the host, not baked into the container. Mount them read-only:

```bash
docker run -it --name devbox \
  -v ~/workspace:/workspace \
  -v ~/.ssh:/home/dev/.ssh:ro \
  -v ~/.gitconfig:/home/dev/.gitconfig:ro \
  devbox /bin/bash
```

Inside the container, authenticate once with:

```bash
gh auth login
claude auth
```

This way you don't accidentally commit credentials to a Docker image, and your auth survives container rebuilds.

### Oh My Posh and Fira Code

[Oh My Posh](https://ohmyposh.dev/) gives you a prompt that actually tells you useful things—git branch, exit codes, execution time. It makes the terminal feel alive. Pair it with [Fira Code](https://github.com/tonsky/FiraCode) (a monospaced font with programming ligatures) in your terminal app for the full effect. Yes, aesthetics matter. If you're going to live in the terminal, make it look good.

### Zellij vs Tmux

[Zellij](https://github.com/zellij-org/zellij) is in the devbox as the terminal multiplexer. For the uninitiated: a multiplexer lets you split your terminal into panes, run multiple sessions, and—crucially—detach and reattach. If your SSH connection drops, your work keeps running.

I started with [tmux](https://github.com/tmux/tmux) because that's what everyone recommends. I'm starting to understand why multiplexers make sense, and I'm still working through the keybindings. Zellij won me over because it shows you the keybindings at the bottom of the screen—very helpful when you're learning. Tmux people will tell me I'm wrong. That's fine.

### iPad Terminal Apps

I use [Blink Shell](https://blink.sh/). It's the best option, but not without quirks:

- Touchscreen text selection works fine
- No touchscreen scrolling though—use keyboard shortcuts
- Tab completion can be flaky
- Occasional input lag when typing fast

**Mosh vs SSH:** [Mosh](https://mosh.org/) handles network interruptions better (great for mobile). SSH is more universally compatible. I use both depending on the connection. If you're on WiFi that drops occasionally, Mosh will save you from losing your session.

### Voice Input

[Superwhisper](https://superwhisper.com/) deserves a mention. Voice-to-text on macOS/iOS that's actually good. Dictate your thoughts, Claude interprets them. There's something satisfying about lying on the couch, talking to your iPad, and watching Claude go ham on your codebase.

### The Devbox Alias

On the host machine, I have an alias to jump straight into the container:

```bash
alias devbox='docker start -ai devbox'
```

SSH into the VPS, type `devbox`, and I'm in. Two keystrokes from anywhere to a fully loaded development environment.

---

## Bonus: VS Code Tunnel

If you still want a GUI escape hatch (no judgment), [VS Code tunnels](https://code.visualstudio.com/docs/remote/tunnels) let you connect VS Code to your VPS from anywhere—including an iPad via [vscode.dev](https://vscode.dev). Run the tunnel on the host (not inside the container), then use VS Code's "Attach to Running Container" feature to get into the devbox with a full editor. Best of both worlds.

---

## Get It

The full Dockerfile, CLAUDE.md, and lazygit config are in a gist:

```bash
git clone https://gist.github.com/BrettKinny/ad8d36a6afee8b32b1ddf482496ffa02 devbox
cd devbox
docker build -t devbox .
docker run -it --name devbox \
  -v ~/workspace:/workspace \
  -v ~/.ssh:/home/dev/.ssh:ro \
  devbox /bin/bash
```

After the first run, reconnect with:

```bash
docker start -ai devbox
```

Clone your repos into `/workspace`, authenticate with `gh auth login` and `claude auth`, and you're off.

---

## What's Next

This is all very fragile but very customisable. The Dockerfile is a starting point—use Claude or Gemini to refine it for your own needs. I find Gemini is particularly good at long-running conversations where you're copy-pasting whole scripts back and forth.

Things I want to explore:

- Swap [superfile](https://github.com/yorukot/superfile) for [yazi](https://github.com/sxyazi/yazi) (Rust-based, reportedly faster)
- Try [Microsoft Edit](https://github.com/microsoft/edit) when it matures
- Maybe go back to tmux or [GNU Screen](https://www.gnu.org/software/screen/) once I properly understand multiplexers
- Add [Open Code](https://github.com/opencode-ai/opencode) as another AI coding option

---

## Wrapping Up

So, I knew what I wanted, and I knew what to do. Sit down with Claude and Gemini and iterate until this thing was humming. It took 27 iterations of the Dockerfile to get here. There were 10+ weird dependency issues that I resolved through pure vibe coding—pasting error logs into the LLM and following instructions until things stopped breaking.

This took a lot more time to get right (and to write this blog post about) than I initially expected. But now I've got a portable, reproducible, disposable development environment that I can access from an iPad on the couch, and Claude can go absolutely ham inside it without risking anything important.

If you build your own version, I'd love to see it. Drop me a line.

Peace. ✌️
