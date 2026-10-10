# xenion0.github.io

Personal blog — penetration testing and application security write-ups.

**Live:** https://xenion0.github.io/

## Stack

- [Astro](https://astro.build) 5 with the [Fuwari](https://github.com/saicaca/fuwari) theme
- Tailwind CSS, Expressive Code, Pagefind search
- Deployed to GitHub Pages by GitHub Actions on every push to `main`

## Requirements

- Node.js 20+
- pnpm 9 — `packageManager` is pinned to `pnpm@9.14.4` and a `preinstall` guard rejects npm/yarn

## Development

```bash
pnpm install
pnpm dev        # http://localhost:4321
pnpm build      # astro build + pagefind index -> dist/
pnpm preview    # serve the production build locally
```

## Writing

| What | Where |
| --- | --- |
| Posts | `src/content/posts/*.md` |
| About page | `src/content/spec/about.md` |
| Site identity, nav, profile, banner | `src/config.ts` |
| Theme styles | `src/styles/` |

Scaffold a post with `pnpm new-post <slug>`.

### Frontmatter

```yaml
---
title: Android Activity Security P2
published: 2022-02-25
description: "One or two sentences shown in listings and meta tags."
image: ''
tags: [Android]
category: Android
draft: false
lang: ''
---
```

`published` must be a date, and `category` a plain string. A post with `draft: true` still renders under `pnpm dev` but is excluded from `pnpm build`.

## Quality checks

```bash
pnpm format   # biome format --write ./src
pnpm lint     # biome check --write ./src
```

`biome ci ./src` runs in CI on every push (`.github/workflows/biome.yml`).

## Deployment

`.github/workflows/deploy.yml` builds with `withastro/action` and publishes to GitHub Pages.
`dist/` is gitignored — never commit it.

## Licensing

- **Theme** (`LICENSE`): MIT, © 2024 saicaca — retained as required.
- **Post content**: CC BY-NC-SA 4.0, configured via `licenseConfig` in `src/config.ts`.
