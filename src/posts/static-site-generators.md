---
title: "Building with Static Site Generators"
description: "Why static sites are making a comeback in modern web development"
date: 2026-01-10
layout: post.njk
---

Static site generators have become increasingly popular for good reason. Let's explore why.

## The Benefits

### Speed
Static HTML files are incredibly fast. No database queries, no server-side rendering - just instant page loads.

### Security
With no server-side code execution or database, the attack surface is minimal. Static sites are inherently more secure.

### Simplicity
The deployment process is straightforward:

1. Build your site locally
2. Upload files to a host
3. Done!

### Cost
Hosting static sites is cheap (or free). Services like GitHub Pages, Netlify, and Vercel offer generous free tiers.

## When to Use Them

Static site generators are perfect for:

- Blogs and documentation
- Portfolio sites
- Marketing pages
- Any content that doesn't change frequently

## The Workflow

My typical workflow looks like this:

```bash
# Start development server
npm start

# Make changes to content

# Build for production
npm run build

# Deploy
git push
```

Simple, efficient, and reliable.

## Popular Options

Some popular static site generators include:

- **11ty** (my choice) - Flexible and simple
- **Hugo** - Blazing fast, written in Go
- **Jekyll** - The classic, GitHub Pages default
- **Gatsby** - React-based, feature-rich
- **Next.js** - Can generate static exports

Each has its strengths, but I've found 11ty hits the sweet spot of power and simplicity.
