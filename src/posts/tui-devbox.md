---
title: "Building a TUI Devbox: A Claude-Powered Terminal Dev Environment running on an iPad"
description: "How I built a portable, reproducible Docker-based terminal development environment packed with modern CLI and TUI tools, designed for Claude Code YOLO mode running from an iPad"
date: 2026-01-28
layout: post.njk
---

*Or how Claude made me stop worrying and love the terminal.*

![Claude Code running on an iPad](/images/claude-ipad.png)

It's 2026 and there's a new UI on the block. Forget React, Angular, Flutter, React Native, MAUI, Electron, SwiftUI... the next bandwagon to jump on is the oldest of them all: the terminal.

I for one am 100% onboard with this trend, and am systematically working towards replacing every piece of software in my life with a TUI. I've been trying to improve my keyboard-only chops slowly for years, and I hate many modern web apps and UIs. Just give me the terminal. Strap me in, and plug me straight into the matrix.

As a Windows kid, the terminal is not my native environment. Sure, I knew how to use DOS to run Wolf3D, but since then I've grown accustomed to the GUI. That's changing now, and bring it on.

This mini project started because I wanted to be able to run [Claude Code](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview) in YOLO mode from my iPad. The iPad doesn't have a proper shell, so I need to SSH into a server. What I set out to do was create a "TUI devbox", a Docker container I can connect to from my iPad (or anywhere) that contains all the tools I need to develop software in a terminal environment.

This article is for:

- People that want to YOLO with Claude in an isolated environment
- People that want to YOLO with Claude on their iPad
- People that are interested in the current crop of CLI/TUI tools
- People that want to dabble in the terminal, just get their toes wet, without having to learn VIM

I'm not going to hold your hand too much. You can ask your favourite LLM about Docker or any acronyms or commands you don't recongise.

---

## Why a Container?

Three reasons:

**Reproducible environment.** I can spin this up anywhere: my home [Unraid](https://unraid.net/) server, my [Hostinger](https://www.hostinger.com/) VPS that I picked up cheap during the Black Friday sales, a mate's machine. Same tools, same config, every time.

**Keeps the host clean.** The container holds the tools. Your projects live outside the container. Blow away the container whenever you want and your work survives. This is the key mental model: the devbox is disposable, the code is not.

**AI loves it.** Claude, Gemini, Codex... they read terminal output natively. Error logs, dependency conflicts, build failures - paste it in and the AI figures it out. An isolated container means you can let it go wild without worrying about it hosing your system and destroying your life.

---

## The Tools

If I'm going to make a TUI devbox, I wanted the best CLI and TUI tools I could find as of January 2026. All the bells and whistles, but not crazy bloated. Modern luxuries like mouse support. I refuse to learn vim.

### Base Image

I started with `ubuntu:24.04`. I could probably optimise this, but I wanted something I know is going to work. No point over-engineering the foundation.

### CLI Tools

| Tool | Language | What It Replaces | Description |
|------|----------|------------------|-------------|
| [fd](https://github.com/sharkdp/fd) | Rust | `find` | Fast, user-friendly alternative |
| [ripgrep (rg)](https://github.com/BurntSushi/ripgrep) | Rust | `grep` | Blazingly fast recursive search |
| [bat](https://github.com/sharkdp/bat) | Rust | `cat` | Syntax highlighting and git integration |
| [zoxide](https://github.com/ajeetdsouza/zoxide) | Rust | `cd` | Learns your habits, jump anywhere |
| [eza](https://github.com/eza-community/eza) | Rust | `ls` | Icons, colours, git status |
| [delta](https://github.com/dandavison/delta) | Rust | `diff` | Syntax-highlighting pager for git diffs |
| [xh](https://github.com/ducaale/xh) | Rust | `curl` | Friendly HTTP client |
| [gh](https://github.com/cli/cli) | Go | - | GitHub's official CLI |
| [yq](https://github.com/mikefarah/yq) | Go | - | YAML/JSON/XML processor |
| [oh-my-posh](https://github.com/JanDeDobbeleer/oh-my-posh) | Go | boring prompt | Cross-shell prompt theme engine |
| [jq](https://github.com/jqlang/jq) | C | - | Lightweight JSON processor |

You'll notice the Rust renaissance here. Basically every classic Unix tool is getting a Rust rewrite that's faster, more user-friendly, and has sane defaults. The Go tools fill in the gaps where Rust hasn't gotten to yet.

### TUI Tools

| Tool | Language | Description |
|------|----------|-------------|
| [zellij](https://github.com/zellij-org/zellij) | Rust | Terminal multiplexer (tmux alternative) |
| [lazygit](https://github.com/jesseduffield/lazygit) | Go | Terminal UI for git |
| [superfile (spf)](https://github.com/yorukot/superfile) | Go | Terminal file manager |
| [micro](https://github.com/zyedidia/micro) | Go | Terminal text editor |
| [fzf](https://github.com/junegunn/fzf) | Go | Interactive fuzzy finder |
| [htop](https://github.com/htop-dev/htop) | C | Interactive process viewer |

---

## How It Fits Together

The *why* matters more than the Dockerfile itself.

### Give Claude Better Tools

When Claude has `ripgrep` instead of `grep`, `fd` instead of `find`, and `bat` instead of `cat`, it writes better commands. It gets syntax-highlighted output. It can parse structured data with `jq` and `yq`. Better tools in, better results out (in theory). 

### Mount Your Code, Don't Copy It

Your projects live on the host, mounted into the container:

```bash
docker run -it --name devbox \
  -v ~/workspace:/workspace \
  devbox /bin/bash
```

Nuke and rebuild the container any time. Your project files don't care.

### No Runtimes Baked In

Working on a Node project? Install Node. .NET? Install the SDK. The devbox gives you the *environment*, not the *stack*.

### Tell Claude What's Installed

[Claude Code](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview) reads a `CLAUDE.md` file for project context. I put one at `~/.claude/CLAUDE.md` listing all the CLI tools in the container, what they do, and the aliases I've set up. Now Claude reaches for `rg` instead of `grep`, `fd` instead of `find`, and so on.

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

### YOLO Mode Needs a Non-Root User

Claude Code has a `--dangerously-skip-permissions` flag that lets it run commands without asking you first. Only works as a non-root user, so the Dockerfile creates a `dev` user for this:

```bash
alias claude-yolo='claude --dangerously-skip-permissions'
```

Claude YOLO on an iPad. What a time to be alive.

**Disclaimer:** I don't know if this environment will enhance Claude Code's performance (speed-wise, probably negligible) or harm it (because these tools might not be in the training data). It seems to be working for me so far..

### Keep Secrets on the Host

`.gitconfig`, SSH keys, and GitHub auth tokens live on the host, not in the container. Mount them read-only:

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

No credentials baked into the image, and auth survives container rebuilds.

### Make the Prompt Useful (and Pretty)

[Oh My Posh](https://ohmyposh.dev/) gives you a prompt that shows git branch, exit codes, and execution time. Pair it with [Fira Code](https://github.com/tonsky/FiraCode) in your terminal app. If you're going to live in the terminal, make it look good.

### Zellij Over Tmux (i'm still deciding)

[Zellij](https://github.com/zellij-org/zellij) is the terminal multiplexer. A multiplexer lets you split your terminal into panes, run multiple sessions, and detach/reattach. Handles SSH drops and session scrolling.

### Blink Shell on iPad

[Blink Shell](https://blink.sh/) is the best option I found. I tried a few and it stood out. Not free.

**Mosh vs SSH:** [Mosh](https://mosh.org/) handles network interruptions better (great for mobile). SSH is more universally compatible. I use both depending on the connection. If you're on WiFi that drops occasionally, Mosh will save you from losing your session.

### Talk to Your iPad

[Superwhisper](https://superwhisper.com/) deserves a mention. Voice-to-text on macOS/iOS/Windows that's actually good. I've been usinmg this to write prompts for Claude Code on the iOS app. I can make changes to this website be talking to my phone.

### Two Keystrokes to Dev Mode

On the host machine:

```bash
alias devbox='docker start -ai devbox'
```

SSH into the VPS, type `devbox`, and I'm in. Two keystrokes from anywhere to a fully loaded development environment.

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

The Dockerfile is a starting point. Use Claude or Gemini to refine it for your needs. Gemini is particularly good at long-running conversations where you're copy-pasting whole scripts back and forth.

Things I want to explore:

- Swap [superfile](https://github.com/yorukot/superfile) for [yazi](https://github.com/sxyazi/yazi) (Rust-based, reportedly faster)
- Maybe go back to tmux or [GNU Screen](https://www.gnu.org/software/screen/) once I properly understand multiplexers
- Add [Open Code](https://github.com/opencode-ai/opencode) as another AI coding option

---

## Wrapping Up

So, I knew what I wanted, and I knew what to do. Sit down with Claude and Gemini and iterate until this thing was humming. It took 27 iterations of the Dockerfile to get here. There were 10+ weird dependency issues that I resolved through pure vibe coding, pasting error logs into the LLM and following instructions until things stopped breaking.

This took a lot more time to get right (and to write this blog post about) than I initially expected. But now I've got a portable, reproducible, disposable development environment that I can access from an iPad on the couch, and Claude can go absolutely ham inside it without risking anything important.

If you build your own, I'm all ears for ideas. Get in touch. 👊s