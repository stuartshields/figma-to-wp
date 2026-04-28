---
name: hero-images
description: Hero image art direction, container heights, and implementation patterns for Primary and Secondary Hero blocks.
disable-model-invocation: true
allowed-tools: Read, Grep, Glob
---

# Skill: hero-images

## When to Use

Use this skill when working on Primary Hero or Secondary Hero blocks, or any block that uses the art-direction-image component with hero-sized containers.

## Art Direction Pattern

Both Primary Hero and Secondary Hero render their images via the shared `template-parts/components/art-direction-image.php` component (`<picture>` + `<img>` with `object-fit: cover`). The same approach is used by non-hero content blocks (ticker-banner, media-split-vertical). There is no `background-image` + `image-set()` pattern in the codebase.

## Hero Container Heights

Both heroes use **fixed heights** at every breakpoint (no `h-dvh`). Each breakpoint has its own token in `assets/theme-variables.css`. The wrapper applies them with base + `sm:` + `lg:` + `2xl:` classes.

**Secondary Hero**

| Breakpoint | Min-width | Token | Height |
|---|---|---|---|
| Mobile | 0 | `--hero-secondary-mobile` | 650 |
| Tablet | 640px | `--hero-secondary-tablet` | 720 |
| Laptop | 1024px | `--hero-secondary-laptop` | 796 |
| Desktop | 1440px+ | `--hero-secondary-desktop` | 796 |

**Primary Hero**

| Breakpoint | Min-width | Token | Height |
|---|---|---|---|
| Mobile | 0 | `--hero-primary-mobile` | 844 |
| Tablet | 640px | `--hero-primary-tablet` | 940 |
| Laptop | 1024px | `--hero-primary-laptop` | 797 |
| Desktop | 1440px+ | `--hero-primary-desktop` | 940 |

## Art Direction Breakpoint

Both heroes use a `640px` art-direction breakpoint: **Mobile** loads the mobile image asset; **Tablet, Laptop, and Desktop** all load the desktop image asset. Editors upload two separate images so the crop/framing differs between portrait-ish (mobile) and landscape (everything else) breakpoints.

## Implementation

- Hero templates pass `desktop_url` and `mobile_url` (built from `wp_get_attachment_image_url()` with the `hero-desktop` / `hero-mobile` image sizes) into `components/art-direction-image`, along with `breakpoint`, `class`, `loading`, and `fetchpriority`.
- The hero `<section>` is a container with the fixed-height tokens applied; the `<picture>` lives inside an absolutely-positioned wrapper with `overflow-clip`. Image uses `object-fit: cover`.
- Alt text flows through to the `<img alt="">` attribute via the component. No `role="img"` on the section.

## Figma BG Image Caveat

Figma hero/banner background images use a **fixed-size container centered** with `left: 50%; top: 50%; transform: translate(-50%, -50%)` (NOT `position: absolute; inset: 0` with `object-cover`). The fixed container keeps zoom/crop constant as the component height changes. Check each breakpoint individually, as desktop may switch to edge-pinned positioning instead of centering.
