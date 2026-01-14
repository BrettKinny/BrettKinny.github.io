# Brett Kinny's Developer Blog

A minimal, TUI-inspired personal developer blog built with [Eleventy (11ty)](https://www.11ty.dev/).

## Design Philosophy

This blog embraces a **Terminal User Interface (TUI)** aesthetic - think command-line interfaces and terminal windows, but refined and minimal. The design features:

- Monospace fonts throughout
- Dark, low-contrast color scheme
- Terminal-style prefixes (`$`, `#`, `>`)
- Clean borders and structured layouts
- No JavaScript, no tracking, no bloat

## Development

### Prerequisites

- Node.js (v14 or higher)
- npm

### Setup

```bash
# Install dependencies
npm install

# Start development server
npm start

# Build for production
npm run build

# Clean build output
npm run clean
```

The development server runs at `http://localhost:8080` with live reload.

## Project Structure

```
.
├── src/                    # Source files
│   ├── _layouts/          # Nunjucks layout templates
│   │   ├── base.njk       # Base layout with header/footer
│   │   └── post.njk       # Blog post layout
│   ├── posts/             # Blog posts (Markdown)
│   │   └── *.md           # Individual blog posts
│   ├── css/               # Source CSS
│   │   └── style.css      # Main stylesheet
│   ├── index.njk          # Homepage template
│   └── posts.njk          # Posts archive template
├── .eleventy.js           # Eleventy configuration
├── package.json           # Project dependencies
└── [build output]/        # Generated files (root directory)
```

## Writing Blog Posts

Create a new Markdown file in `src/posts/` with front matter:

```markdown
---
title: "Your Post Title"
description: "A brief description"
date: 2026-01-14
layout: post.njk
---

Your post content here...
```

Posts are automatically:
- Added to the posts collection
- Sorted by date (newest first)
- Listed on the homepage and posts archive

## Deployment

This site is configured for GitHub Pages:

1. Build the site: `npm run build`
2. Commit all changes (including built files in root)
3. Push to GitHub
4. GitHub Pages will serve from the root directory

## Customization

- **Colors**: Edit CSS variables in `src/css/style.css`
- **Fonts**: Modify the font-family in `src/css/style.css`
- **Header**: Update `src/_layouts/base.njk`
- **Build output**: Change `dir.output` in `.eleventy.js`

## Tech Stack

- **[Eleventy](https://www.11ty.dev/)**: Static site generator
- **[Nunjucks](https://mozilla.github.io/nunjucks/)**: Templating language
- **Markdown**: Content format
- **CSS**: Styling (no frameworks)

## License

ISC
