---
title: "Hello World"
description: "What this blog is about and how it's built"
date: 2026-01-05
layout: post.njk
---

```c
printf("hello, world\n");
```

This is my personal blog. I started it because I wanted a place to document what I'm learning and building — mostly for myself, but if someone else finds it useful, even better.

I'll write about industrial automation, the tools I use, and whatever else.

## Tech stack

This site is built with [Eleventy](https://www.11ty.dev/) (11ty), a static site generator. Markdown files go in, HTML comes out. No database, no server-side code.

- **Static site generator:** [Eleventy v3](https://www.11ty.dev/)
- **Feed:** RSS via [@11ty/eleventy-plugin-rss](https://www.11ty.dev/docs/plugins/rss/)
- **Hosting:** GitHub Pages

The design is minimal on purpose. No tracking, no analytics, no bullshit. It loads fast because there's nothing to load.

## What's next

First up will probably be some posts on PLC programming and industrial automation. After that, who knows.
