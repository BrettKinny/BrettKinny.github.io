---
title: "Building My TUI Devbox: A Containerised Development Environment"
description: "A Docker-based terminal playground with modern CLI tools and AI assistance. Turn your iPad into a portable dev machine."
date: 2026-01-16
layout: post.njk
---

I built a Docker-based development environment packed with modern CLI tools. Here's what I learned—and why AI made the terminal fun again.

## Why Build This?

I wanted to code on my iPad. Not in some watered-down web IDE, but a proper terminal environment with real tools. The new iPad OS with proper windowing and Stage Manager makes this more viable than ever. Combine that with voice input and a good terminal app, and suddenly the iPad isn't a compromise—it's genuinely useful.

But more than that, I wanted a reproducible environment. Something I could spin up anywhere—my iPad over SSH, a cloud VM, my laptop. Containers make you ambidextrous. Local or cloud, it doesn't matter. Your environment is the same.

Setting this up with Claude and Gemini was genuinely fun. The AI didn't just write configs—it gave me a reason to explore tools I'd been meaning to try for years.

## What's Inside

The goal was modern replacements for crusty old Unix tools. Stuff that's actually pleasant to use:

- **fd, rg, bat, fzf** - Modern replacements for find, grep, cat. Faster, prettier, better defaults.
- **eza** - `ls` with icons and git status. Requires a Nerd Font (I use [Fira Code](https://www.nerdfonts.com/font-downloads)).
- **lazygit + delta** - TUI git client with syntax-highlighted diffs. Game changer.
- **zellij** - Terminal multiplexer. Like tmux but actually learnable.
- **zoxide** - Smart `cd` that learns your habits. Type `z proj` and it figures out you mean `/home/dev/workspace/my-project`.
- **xh** - HTTPie alternative for testing APIs.
- **superfile** - TUI file manager. Navigate with arrow keys like a normal person.
- **micro** - Simple terminal editor. I hate nano, and I'm not a vim person. Microsoft's new [Edit](https://github.com/microsoft/edit) is worth a look too.
- **oh-my-posh** - Pretty prompts that show git status, environment info, etc.
- **GitHub CLI** - `gh pr create` from the terminal. No browser needed.
- **Claude Code** - The real star. AI coding assistant that lives in your terminal.

BYO SDK—the container is intentionally minimal. Mount your project folder and install what you need for that specific project.

## The Tricky Bits

### Running Claude in YOLO Mode

Claude Code has a "dangerously skip permissions" mode that lets it run commands without asking. Essential for flow state. But it only works when running as a non-root user. The Dockerfile creates a `dev` user specifically for this:

```bash
echo "alias claude-yolo='claude --dangerously-skip-permissions'" >> ~/.bashrc
source ~/.bashrc
```

YOLO mode on an iPad. What a time to be alive.

### The Workspace Mount

Don't put your code inside the container. Mount an external folder:

```bash
docker run -it --name devbox -v ~/workspace:/workspace devbox /bin/bash
```

Your code lives on the host (or in persistent storage), the tools live in the container. Blow away the container whenever you want—your work survives.

### iPad Terminal Apps

I use [Blink Shell](https://blink.sh/). It's the best option, but not without quirks:

- Touchscreen text selection works fine
- No touchscreen scrolling though—use keyboard shortcuts
- Tab completion can be flaky
- Occasional input lag when typing fast

Mosh vs SSH: Mosh handles network interruptions better (great for mobile), but SSH is more universally compatible. I use both depending on the connection.

### Voice Input Changes Everything

WhisperFlow and similar tools make voice-to-text actually usable. Combined with Claude Code understanding natural language, you can describe what you want and watch it happen. The iPad keyboard becomes optional.

## Get It

```bash
git clone https://gist.github.com/BrettKinny/ad8d36a6afee8b32b1ddf482496ffa02 devbox
cd devbox
docker build -t devbox .
docker run -it --name devbox -v ~/workspace:/workspace devbox /bin/bash
```

After the first run, reconnect with:

```bash
docker start -ai devbox
```

## How I Write This Blog

Speaking of workflow—this post was written in the devbox:

1. Draft in Obsidian (or just vim the markdown directly)
2. Claude Code generates the post and handles frontmatter
3. 11ty builds static HTML
4. Push to GitHub, Pages deploys automatically

No CMS, no database, no drama.

## What's Next

Things I'd improve:

- Better Neovim config for when I'm feeling brave
- Investigate Dev Containers spec for IDE integration
- Maybe try Helix editor
- More language-specific SDK containers

The whole point is that it's a playground. Break things, rebuild, learn something.

---

[Grab the Dockerfile from the gist](https://gist.github.com/BrettKinny/ad8d36a6afee8b32b1ddf482496ffa02)
