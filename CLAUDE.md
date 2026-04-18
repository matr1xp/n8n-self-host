# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This is a **static marketing/landing page** for an n8n self-hosted automation service, deployed via GitHub Pages at `n8n.ml1.app`. It is not an n8n deployment configuration — it is simply the public-facing website for an n8n instance hosted elsewhere.

## Repository Structure

```
index.html           # Main landing page
privacy_policy.html  # Privacy policy page
CNAME                # GitHub Pages custom domain (n8n.ml1.app)
.github/workflows/
  static.yml         # Auto-deploy to GitHub Pages on push to master
```

## Deployment

There is no build step. Changes pushed to the `master` branch are automatically deployed to GitHub Pages via the workflow in `.github/workflows/static.yml`.

- **Live URL**: https://n8n.ml1.app
- **Branch to deploy**: `master`
- **Deployment**: Automated — the entire repository is uploaded as a Pages artifact

To deploy: commit changes and push to `master`. GitHub Actions handles the rest.

## Styling Conventions

- **CSS framework**: Tailwind CSS loaded via CDN (`https://cdn.tailwindcss.com`) — no build step required
- **Font**: Inter, loaded from `https://rsms.me/inter/inter.css`
- **Color scheme**: Dark theme using `bg-gray-900` (page background) and `bg-gray-800` (card/section backgrounds)
- **Accent color**: Indigo-600 for primary CTAs, white for secondary CTAs
- **Layout**: `max-w-3xl` for centered text sections, `max-w-7xl` for full-width grid sections
- **Cards**: `rounded-xl shadow-lg` with hover state `hover:shadow-2xl` for feature cards in sections on gray-900 background; flat `rounded-xl shadow-lg` (no hover) for cards on gray-800 background

Prefer Tailwind utility classes over custom CSS. Custom CSS belongs in the `<style>` block in `<head>` only when Tailwind cannot handle it.

## Page Structure Pattern

Both HTML pages share the same structure: Inter font + Tailwind CDN in `<head>`, `bg-gray-900 text-gray-200` on `<body>`, and a footer with the copyright and privacy policy link.
