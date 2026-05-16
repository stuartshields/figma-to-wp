<!-- Last updated: 2026-05-16T18:00+10:00 -->

# Figma → Code Token Map

Generated from `content/themes/wp-theme/theme.json`, `theme-variables.css`, and `DESIGN-TOKENS.md`. Re-run via `design-tokens/SKILL.md` when tokens change in Figma.

**This file is the canonical desktop ↔ mobile spacing reference.** The figma-workflow skill and `DESIGN-TOKENS.md` both link here rather than duplicate the table.

**The canonical reference is `content/themes/wp-theme/DESIGN-TOKENS.md`**. Read it for context and the why behind the scale. This file is the lookup table: when Figma returns a variable name, find it here and use the matching token.

---

## Critical notes before using any token

1. **This theme is Tailwind 4.** It uses arbitrary CSS variable syntax like `max-w-(--layout-site)`, `bg-(--color-vanilla)`, `rounded-(--radius-card)`. Do **not** use Tailwind 3 `max-w-[var(--layout-site)]` syntax. It will silently break.
2. **Spacing tokens have different values at mobile vs desktop.** Figma's `spacing/150` means 12px on desktop but 8px on mobile. Our `theme.json` only holds desktop values. See the mobile-vs-desktop section below and **do not skip it**.
3. **Never hardcode hex values or raw pixel measurements.** If Figma returns `#f2796c`, that is the `peach` token. Use `bg-peach` / `text-peach` / `--color-peach`, never `bg-[#f2796c]`.
4. **Two parallel systems exist.** `theme.json` drives the block editor and WP Global Styles (`var(--wp--preset--color--peach)`). `theme-variables.css` drives Tailwind utilities (`bg-peach`, `gap-150`). They hold the same values. When a token changes in Figma, update both.

---

## Colours

Figma uses semantic names. Our palette mirrors them 1:1. **No translation needed**, the Figma name IS the token name.

| Figma name | Hex | Tailwind utility | CSS variable | theme.json slug |
|---|---|---|---|---|
| Vanilla | `#fffcf5` | `bg-vanilla` `text-vanilla` | `--color-vanilla` | `vanilla` |
| Cream | `#fff7ef` | `bg-cream` | `--color-cream` | `cream` |
| Plum 700 | `#3b0e1c` | `bg-plum-700` `text-plum-700` | `--color-plum-700` | `plum-700` |
| Plum 600 | `#52172a` | `bg-plum-600` | `--color-plum-600` | `plum-600` |
| Plum 500 | `#6b2138` | `bg-plum-500` | `--color-plum-500` | `plum-500` |
| Plum 300 | `#9f6874` | `bg-plum-300` | `--color-plum-300` | `plum-300` |
| Peach | `#f2796c` | `bg-peach` | `--color-peach` | `peach` |
| Peach Hover | `#fba79c` | `bg-peach-hover` | `--color-peach-hover` | `peach-hover` |
| Melon | `#ffdccd` | `bg-melon` | `--color-melon` | `melon` |
| Melon Light | `#ffede3` | `bg-melon-light` | `--color-melon-light` | `melon-light` |
| Salmon | `#f0c6b4` | `bg-salmon` | `--color-salmon` | `salmon` |
| Toffee | `#dfab95` | `bg-toffee` | `--color-toffee` | `toffee` |
| Ice | `#bcd7ff` | `bg-ice` | `--color-ice` | `ice` |
| Ice Light | `#cfe5ff` | `bg-ice-light` | `--color-ice-light` | `ice-light` |
| Sea Salt | `#e7f2ff` | `bg-sea-salt` | `--color-sea-salt` | `sea-salt` |
| Berry | `#153152` | `bg-berry` | `--color-berry` | `berry` |
| Navy | `#041b36` | `bg-navy` | `--color-navy` | `navy` |
| White | `#ffffff` | `bg-white` | `--color-white` | `white` |
| Text Secondary | `#3b3b3b` | `text-text-secondary` | `--color-text-secondary` | `text-secondary` |
| Gray 200 | `#e5e5e5` | `bg-gray-200` | `--color-gray-200` | `gray-200` |
| Gray 300 | `#d1d4d9` | `bg-gray-300` | `--color-gray-300` | `gray-300` |
| Gray 500 | `#999999` | `bg-gray-500` | `--color-gray-500` | `gray-500` |
| Border | `#eff0f2` | `border-border` | `--color-border` | `border` |

**Also in `theme-variables.css` but not in theme.json** (front-end only, no editor preset):
- `--color-melon-dark: #ffe3d6` → `bg-melon-dark`
- `--color-ice-dark: #b5d7ff` → `bg-ice-dark`

### Semantic colour aliases (`:root` in theme-variables.css)

Prefer these when Figma uses semantic intent rather than a specific colour:

| Semantic | CSS variable | Resolves to |
|---|---|---|
| Primary surface | `--surface-primary` | `vanilla` |
| Secondary surface | `--surface-secondary` | `sea-salt` |
| Tertiary surface | `--surface-tertiary` | `melon` |
| Dark primary surface | `--surface-invert-primary` | `plum-500` |
| Dark secondary surface | `--surface-invert-secondary` | `navy` |
| Primary text | `--text-primary` | `plum-700` |
| Secondary text | `--text-secondary` | `#3b3b3b` |
| Accent text | `--text-accent` | `plum-500` |
| Invert text (on dark) | `--text-primary-invert` | `white` |
| CTA surface | `--cta-primary-surface` | `peach` |
| CTA text | `--cta-primary-text` | `plum-700` |
| CTA outline border (on dark) | `--cta-outline-border` | `rgba(255,255,255,0.2)` |
| CTA outline text (on dark) | `--cta-outline-text` | `white` |

---

## Spacing: Desktop

Scale uses a numbered system. **The number is roughly px × 12.5**, but don't do the math. Use the table. Tokens work with every spacing utility: `gap-`, `p-`, `px-`, `py-`, `m-`, `mx-`, `my-`, `pt-`, etc.

| Figma | Desktop value | Token |
|---|---|---|
| `spacing/50` | 4px | `gap-50` / `p-50` / etc. |
| `spacing/62` | 5px | `gap-62` |
| `spacing/75` | 6px | `gap-75` (css-only, not in theme.json) |
| `spacing/100` | 8px | `gap-100` |
| `spacing/125` | 10px | `gap-125` |
| `spacing/150` | 12px | `gap-150` |
| `spacing/175` | 14px | `gap-175` |
| `spacing/200` | 16px | `gap-200` |
| `spacing/225` | 18px | `gap-225` |
| `spacing/250` | 20px | `gap-250` |
| `spacing/300` | 24px | `gap-300` |
| `spacing/325` | 26px | `gap-325` |
| `spacing/350` | 28px | `gap-350` (css-only) |
| `spacing/375` | 30px | `gap-375` |
| `spacing/400` | 32px | `gap-400` |
| `spacing/500` | 40px | `gap-500` |
| `spacing/537` | 43px | `gap-537` |
| `spacing/600` | 48px | `gap-600` |
| `spacing/750` | 60px | `gap-750` (css-only) |
| `spacing/800` | 64px | `gap-800` |
| `spacing/1000` | 80px | `gap-1000` |
| `spacing/1200` | 96px | `gap-1200` |
| `spacing/1500` | 120px | `gap-1500` |

**This is not an 8pt grid.** Values like 5px, 10px, 14px, 30px, 43px, 60px are intentional. Do not "round" them or flag them as inconsistent.

**Use tokens over raw Tailwind multipliers.** Write `gap-150` (12px) not `gap-3` (12px). They produce the same CSS, but `gap-150` ties back to the design system.

---

## Spacing: Mobile ≠ Desktop (critical)

Figma's spacing variables have two modes. The same variable name resolves to different pixel values. `theme.json` only holds the **desktop** values because WordPress doesn't support responsive spacing presets. So when Figma says `spacing/150` at mobile, that is **not** `gap-150` (which is 12px). It's 8px, which is `gap-100` on our scale.

| Figma var | Desktop value | Desktop token | Mobile value | Mobile token |
|---|---|---|---|---|
| `spacing/50` | 4px | `*-50` | 2px | `*-0.5` |
| `spacing/150` | 12px | `*-150` | 8px | `*-100` |
| `spacing/200` | 16px | `*-200` | 12px | `*-150` |
| `spacing/250` | 20px | `*-250` | 16px | `*-200` |
| `spacing/300` | 24px | `*-300` | 16px | `*-200` |
| `spacing/400` | 32px | `*-400` | 20px | `*-250` |
| `spacing/500` | 40px | `*-500` | 24px | `*-300` |

### How to apply

**Mobile-only templates** (anything inside `lg:hidden` or the mobile menu panel). Use the mobile token directly:
```html
<div class="gap-100"><!-- Figma spacing/150 at mobile = 8px --></div>
```

**Desktop-only templates** (anything inside `hidden lg:flex`). Use the desktop token:
```html
<div class="gap-150"><!-- Figma spacing/150 at desktop = 12px --></div>
```

**Shared / responsive elements.** Mobile token as the base, desktop with `lg:` prefix:
```html
<div class="gap-100 lg:gap-150"><!-- 8px mobile, 12px desktop --></div>
```

We did not create separate `-mobile` spacing tokens because the mobile values already exist at other positions on the scale. Duplicating would bloat the token count with no benefit.

---

## Typography

**One font family throughout: Neulis Neue.** Loaded from `assets/fonts/neulis-neue/`.

### Weights

| Weight | Value | Usage |
|---|---|---|
| Regular | 400 | Body text, nav links |
| Medium | 500 | Buttons, bold body |
| Semi-bold | 600 | Headings |
| Extra Bold | 800 | Eyebrow labels (uppercase) |

### Font sizes: desktop

| Figma name | Value | Token | theme.json slug |
|---|---|---|---|
| Display | 72px | `text-display` | `display` |
| H1 | 56px | `text-h1` | `h1` |
| H2 | 48px | `text-h2` | `h2` |
| H3 | 36px | `text-h3` | `h3` |
| H4 | 28px | `text-h4` | `h4` |
| Intro | 24px | `text-intro` | (css-only) |
| Large Body | 20px | `text-lg-body` | `lg-body` |
| Body | 18px | `text-body` | `body` |
| Large Button | 18px | `text-lg-button` | `lg-button` |
| Small Body | 16px | `text-sm-body` | `sm-body` |
| Button | 16px | `text-button` | `button` |
| Eyebrow | 14px | `text-eyebrow` | `eyebrow` |
| Caption | 14px | `text-caption` | (css-only) |

### Font sizes: mobile

Typography **does** get dedicated `-mobile` tokens (unlike spacing), because mobile type values don't map cleanly to other desktop positions. **Always use a `-mobile` token for mobile type. Never repurpose a desktop token that happens to share the same pixel value.**

| Mobile token | Value | Desktop equivalent |
|---|---|---|
| `text-display-mobile` | 40px | `text-display` (72px) |
| `text-h3-mobile` | 36px | `text-h1` (56px) |
| `text-h2-mobile` | 32px | `text-h2` (48px) |
| `text-h4-mobile` | 24px | `text-h4` (28px) |
| `text-intro-mobile` | 20px | `text-intro` (24px) |
| `text-lg-body-mobile` | 18px | `text-lg-body` (20px) |
| `text-body-mobile` | 16px | `text-body` (18px) |
| `text-sm-body-mobile` | 14px | `text-sm-body` (16px) |
| `text-caption-mobile` | 12px | `text-caption` (14px) |

### Font sizes: tablet

| Tablet token | Value | Desktop equivalent |
|---|---|---|
| `text-display-tablet` | 60px | `text-display` (72px) |
| `text-h2-tablet` | 36px | `text-h2` (48px) |
| `text-h4-tablet` | 26px | `text-h4` (28px) |

### Line heights

Line heights are defined separately from font sizes and must be paired explicitly.

**Desktop:** `leading-display` 72px, `leading-h1` 60px, `leading-h2` 52px, `leading-h3` 44px, `leading-h4` 36px, `leading-intro` 32px, `leading-body` 28px, `leading-lg-body` 28px, `leading-sm-body` 24px, `leading-button` 24px, `leading-lg-button` 24px, `leading-eyebrow` 20px, `leading-caption` 20px.

**Tighter body** (comments, compact text blocks): `leading-body-tight` 26px.

**Tablet:** `leading-display-tablet` 60px, `leading-h2-tablet` 40px.

**Mobile:** `leading-display-mobile` 40px, `leading-h3-mobile` 40px, `leading-h2-mobile` 36px, `leading-h4-mobile` 32px, `leading-intro-mobile` 28px, `leading-lg-body-mobile` 26px, `leading-body-mobile` 24px.

### Letter spacing

| Token | Value | Usage |
|---|---|---|
| `tracking-eyebrow` | 2px | Uppercase eyebrow labels |

---

## Border Radius

| Token | Value | Tailwind utility | Usage |
|---|---|---|---|
| `--radius-card` | 16px | `rounded-(--radius-card)` or `rounded-card` | Cards (desktop) |
| `--radius-card-mobile` | 8px | `rounded-card-mobile` | Cards (mobile) |
| `--radius-image` | 16px | (none) | Images (desktop) |
| `--radius-image-mobile` | 8px | (none) | Images (mobile) |
| `--radius-tile` | 20px | (none) | Ticker digit tiles |
| `--radius-button` | 1000px | `rounded-(--radius-button)` / `rounded-full` | Pill buttons |

**Additionally from theme.json `settings.custom.radius`:**
- `sm`: 2px, small radius
- `button`: 1000px, mirror of `--radius-button`
- `card`: 16px / `cardMobile`: 8px, mirror of card radii
- `image`: 16px / `imageMobile`: 8px, mirror of image radii

---

## Shadows

| Token | Value | Usage |
|---|---|---|
| `--shadow-picture` | `0 4px 40px rgba(107, 33, 56, 0.15)` (plum, 15%) | Featured images / picture cards |

---

## Blur (backdrop)

Used for hero overlays and glass-morphism (ticker-banner glass tiles, modal overlays).

| Token | Value |
|---|---|
| `--blur-light` | 6px |
| `--blur-medium` | 20px |
| `--blur-heavy` | 25px |
| `--blur-max` | 30px |

Also in `theme.json` under `settings.custom.blur`: `light` / `medium` / `heavy` / `max` (same values).

---

## Layout Widths

| Token | Value | Usage |
|---|---|---|
| `--layout-site` | 1440px | Max site width (outer container) |
| `--layout-wide` | 1792px | Wide-align content max |
| `--layout-content` | 820px | Default content column (matches `theme.json` `contentSize`) |
| `--layout-content-narrow` | 680px | Narrow content column |
| `--layout-modal` | 1200px | Modal max-width |
| `--layout-heading-max` | 572px | Max width for heading text (prevents ultra-wide headings) |
| `--layout-mobile-min` | 375px | Minimum block width (matches Figma mobile frame) |

`theme.json` `contentSize: 820px`, `wideSize: 1312px`. Use these via Tailwind arbitrary-variable syntax: `max-w-(--layout-site)`, `max-w-(--layout-content)`, etc.

---

## Grid

Theme-level grid config (from `theme.json` `settings.custom.grid`):

- Screen size: **1440px**
- Margin: **64px**
- Gutter: **20px**
- Columns: **12**

CSS variables (`:root`): `--grid-screen: 1440px`, `--grid-margin: 64px`, `--grid-gutter: 20px`.

---

## Component padding tokens

From `theme.json` `settings.custom.padding` and mirrored in `theme-variables.css`:

| Token | Value | Usage |
|---|---|---|
| `--padding-component-h-sm` | 40px | Small horizontal component padding |
| `--padding-component-h-md` | 64px | Medium horizontal component padding |
| `--padding-component-v-sm` | 40px | Small vertical component padding |
| `--padding-component-v-md` | 64px | Medium vertical component padding |
| `--padding-component-v-lg` | 96px | Large vertical component padding |
| `--padding-component-v-xl` | 120px | Extra large vertical component padding |

---

## Component-specific dimensions

These are hand-tuned dimensions from Figma for specific components. Not generic tokens. Use only in the component they belong to. Full list is in `theme-variables.css`; this is a topical index.

**Hero heights:** `--hero-primary-mobile` (844px), `--hero-primary-tablet` (940px), `--hero-primary-laptop` (797px), `--hero-primary-desktop` (940px), `--hero-secondary-mobile` (650px), `--hero-secondary-tablet` (720px), `--hero-secondary-laptop` / `-desktop` (796px), `--hero-min-mobile` (400px), `--hero-min-desktop` (600px), `--hero-padding-top` (140px).

**Card image heights:** `--card-image-sm` (240px), `--card-image-md` (300px), `--card-image-lg` (420px), `--card-image-short` (235px).

**People/profile cards:** `--card-people` (400px), `--card-people-mobile-width` (300px), `--card-people-tablet` (360px), `--card-people-tablet-height` (480px), `--card-people-desktop` (420px).

**Carousel heights:** `--carousel-height-mobile` (484px), `--carousel-height-tablet` (570px), `--carousel-height-desktop` (696px), `--carousel-card-offset` (32px), `--carousel-card-full: calc(100vw - 32px)`, `--padding-carousel-bottom` (34px).

**Service cards:** `--card-service` (420px), `--card-service-desktop` (600px), `--card-service-w` (460px), `--card-service-min` (690px).

**Article / feature images:** `--image-article` (480px), `--image-article-lg` (640px), `--image-feature` (560px).

**Icons:** `--icon-button` (52px), `--icon-lg` (72px), `--icon-touch` (44px), `--size-icon-lg` (72px).

**Modal (team):** `--modal-image-w` (380px), `--modal-image-h` (460px), `--modal-content-min` (300px), `--modal-max-h: calc(100dvh - 72px)`, `--modal-max-h-padded`, `--modal-max-h-desktop: calc(100dvh - 128px)`.

**Info grid (USP):** `--usp-primary-h` (400px), `--usp-primary-h-tablet` (480px), `--usp-card-h` (356px), `--usp-card-w` (424px), `--usp-heart-*` (decorative heart positioning and size), `--usp-graphic-w` (982px), `--usp-graphic-h` (978px).

**Ticker banner:** `--ticker-height-mobile` (650px), `--ticker-height-tablet` (700px), `--ticker-height` (780px), `--ticker-gradient-height` (660px), `--ticker-tile-w-mobile` (50px), `--ticker-tile-w-tablet` (89px), `--ticker-tile-w` (112px).

**Footer:** `--footer-ornament` (169px), `--footer-ornament-desktop` (194px), `--footer-logo-w` (189px), `--footer-logo-h` (50px), `--footer-logo-w-desktop` (212px), `--footer-logo-h-desktop` (56px), `--footer-brand-col` (260px).

**Navigation:** `--nav-dropdown-w` (619px), `--nav-dropdown-offset` (36px), `--nav-dropdown-min-w` (200px).

**Full-width image overlap layout:** `--overlap-pb` (352px), `--overlap-mb` (304px), `--overlap-pr` (368px), `--overlap-card-w` (656px).

**Media split min-height:** `--media-split-min` (806px), `--media-split-max-w` (896px), `--media-split-image-mobile` (400px).

**Form fields:** `--form-field-min` (280px), `--form-col-min` (320px), `--form-max-w` (420px).

**Recipe / content:** `--recipe-dropdown-w` (336px), `--recipe-info-label` (227px), `--recipe-toggle-w` (102px).

**Content overflow:** `--content-overflow` (232px), `--content-overflow-desktop` (287px), `--content-overflow-ms` (72px), `--content-overflow-ms-desktop` (85px).

**Comment gaps:** `--gap-comment-sm` (36px), `--gap-comment-lg` (72px).

**Dots & dividers:** `--line-weight` (2px), `--dot-size` (6px), `--dot-gap` (2px), `--divider-height` (11px), `--dot-offset-active` (9px), `--dot-offset` (11px).

**Component-specific paddings:** `--padding-team-top` (60px), `--padding-contact-top` (160px).

**Nudges:** `--align-checkbox` (2px).

**Hover underline expansion:** `--underline-expand: calc(100% + 32px)` (extends 32px beyond link text).

---

## Breakpoint overrides

| Breakpoint | Value | Notes |
|---|---|---|
| `--breakpoint-2xl` | 90rem (1440px) | Matches Figma desktop frame |
| `--breakpoint-3xl` | 120rem (1920px) | Widescreen |

Standard Tailwind breakpoints (`sm`, `md`, `lg`, `xl`) are unchanged.

---

## Tailwind 4 syntax reminders

This theme is on **Tailwind 4**. When referencing CSS variables from Tailwind utilities, use the arbitrary CSS variable shorthand:

```html
<!-- Correct (Tailwind 4) -->
<div class="max-w-(--layout-site) bg-(--color-vanilla) h-(--hero-primary-desktop)">

<!-- Wrong (Tailwind 3 legacy, do not use) -->
<div class="max-w-[var(--layout-site)] bg-[var(--color-vanilla)]">
```

Both technically work, but Tailwind 4 parentheses syntax is the project convention. Preserve it when editing existing templates.
