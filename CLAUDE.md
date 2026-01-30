# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

8-Bit Labs is a Jekyll-based static blog about retro computing, 8-bit microprocessors (6502, Z80, 8080), and vintage gaming hardware. The site is hosted on GitHub Pages at https://8bitlabs.ca.

## Development Commands

```bash
# Install dependencies
bundle install

# Serve locally with live reload (primary development command)
bundle exec jekyll serve -l

# Site available at http://localhost:4000
```

GitHub Pages automatically builds and deploys on push to main.

## Creating Blog Posts

Posts go in `_posts/` with filename format `YYYY-MM-DD-title.md`:

```yaml
---
layout: post
published: true
date: 2024-01-01 00:00:00 -0500
title: My Post Title
tags: ["6502", "z80"]
---
Content in Markdown...
```

- Drafts go in `_drafts/` (see `_drafts/template.md`)
- Valid tags correspond to files in `Tags/` directory: 6502, 8080, z80, agon, c64, eater, gameboy, rc2014
- Images go in `assets/img/`, downloadable files in `assets/files/`
- Permalink format: `/Posts/YYYY/MM/DD/title`

## Architecture

- **Layouts**: `_layouts/default.html` (base) → `post.html` (posts), `tag.html` (tag archives)
- **Includes**: `_includes/` has nav.html, head.html, pagination.html, analytics.html
- **Styling**: PicoCSS framework (`css/pico.css`) + custom (`css/site.css`, `css/code.css`)
- **Theme switcher**: `js/minimal-theme-switcher.js` handles dark/light/auto modes

## Configuration

Key settings in `_config.yaml`:
- Pagination: 8 posts per page
- Syntax highlighting: Rouge with Kramdown markdown
- Timezone: America/New_York
