<!-- Last updated: 2026-05-16T17:00+10:00 -->

# Design Tokens

How the design system connects Figma to the front end, and what you need to know to use it.

> **Lookup table lives in `.claude/figma-token-map.md`.** This file documents the *why*; the map is the *what*. The Figma → token pipeline is documented in `design-tokens/SKILL.md` (10up's exporter plugin, theme.json canonical, Tailwind `@theme` aliases).

## Token Flow

Figma variables are the source of truth. 10up's `figma-to-wordpress-theme-json-exporter` plugin emits a `theme.json` fragment under `settings.custom.*`. That's the canonical landing zone, because nesting under `settings.custom` is what gives WordPress permission to emit project-specific CSS variables (`--wp--custom--<group>--<name>`). The well-known top-level `settings.color` / `settings.spacing` / `settings.typography` (which emit `--wp--preset--*` variables and populate the editor's colour/spacing pickers) are written by the exporter too when a Figma collection maps cleanly to a WP preset bucket, but most project tokens live under `custom`.

```
Figma variables
    |
    +-- theme.json
    |     settings.color.palette       --> --wp--preset--color--peach    (editor presets + Global Styles)
    |     settings.spacing.spacingSizes --> --wp--preset--spacing--150   (editor presets + Global Styles)
    |     settings.custom.color.peach   --> --wp--custom--color--peach   (project-specific tokens)
    |     settings.custom.spacing.150   --> --wp--custom--spacing--150
    |
    +-- assets/theme-variables.css (@theme)
          --color-peach: var(--wp--custom--color--peach);
          --spacing-150: var(--wp--custom--spacing--150);
          (Tailwind utilities: gap-150, bg-peach)
```

`theme.json` drives the block editor and WP's Global Styles. `theme-variables.css` aliases the WP custom properties to short Tailwind-friendly names used in templates and static HTML. There is one underlying value per token; the Tailwind file is an alias layer, not a duplicate definition.

If a token changes in Figma, re-run the exporter (see `design-tokens/SKILL.md`) and then refresh the alias file and the maps.

## Spacing Scale

The spacing tokens use a numbered scale. The number is roughly the pixel value multiplied by 12.5, but don't try to do the math - just use the table.

| Token | Value | Tailwind utility |
|-------|-------|------------------|
| `50` | 4px | `gap-50`, `p-50`, `m-50` |
| `62` | 5px | `gap-62` |
| `100` | 8px | `gap-100`, `p-100` |
| `125` | 10px | `gap-125`, `p-125` |
| `150` | 12px | `gap-150`, `p-150` |
| `175` | 14px | `gap-175` |
| `200` | 16px | `gap-200`, `p-200` |
| `225` | 18px | `gap-225` |
| `250` | 20px | `gap-250`, `p-250` |
| `300` | 24px | `gap-300`, `p-300` |
| `325` | 26px | `gap-325` |
| `375` | 30px | `gap-375` |
| `400` | 32px | `gap-400`, `p-400` |
| `500` | 40px | `gap-500`, `p-500` |
| `537` | 43px | `gap-537` |
| `600` | 48px | `gap-600` |
| `800` | 64px | `gap-800`, `px-800` |
| `1000` | 80px | `gap-1000` |
| `1200` | 96px | `gap-1200` |
| `1500` | 120px | `gap-1500` |

This is not an 8pt grid. The scale follows Figma's spacing variables directly, so values like 5px, 10px, 14px, 30px, and 43px are intentional. Don't snap tokens to an 8pt grid or flag them as inconsistent.

These tokens work with any spacing utility - `gap-`, `p-`, `px-`, `py-`, `m-`, `mx-`, `my-`, `pt-`, etc.

Use tokens instead of raw Tailwind multipliers. Write `gap-150` (12px) not `gap-3` (12px). Both produce the same CSS but `gap-150` ties back to the design system.

## Mobile vs Desktop Spacing

This is the part that catches people out.

Figma's spacing variables have two modes - desktop and mobile. The same variable name (`spacing/150`) resolves to different pixel values depending on the mode. Our `theme.json` and `theme-variables.css` only hold the **desktop** values, because WordPress doesn't support responsive spacing presets.

So when you're building a mobile layout and Figma says `spacing/150`, you can't use `gap-150`. That's the desktop value (12px). The mobile value for that same Figma variable is 8px, which is `gap-100` on our scale.

**The mapping table lives in `.claude/figma-token-map.md` under "Spacing: Mobile ≠ Desktop."** That is the single source of truth. Applying it in templates is what the figma-workflow skill does on every Figma session.

### Why the names don't match

Because WordPress `theme.json` doesn't support responsive spacing. One value per token. We use the desktop values since that's what the block editor renders at. Mobile differences are handled in templates using different tokens from the same scale.

We didn't create separate `-mobile` tokens because the mobile values already exist at other positions on the scale. Adding duplicates would double the token count for no benefit.

## Typography

One font family throughout - Neulis Neue.

| Weight | Value | Usage |
|--------|-------|-------|
| Regular | 400 | Body text, nav links |
| Medium | 500 | Buttons, bold body |
| Semi-bold | 600 | Headings |
| Extra Bold | 800 | Eyebrow labels |

Font sizes have dedicated tokens (`text-h2`, `text-body`, `text-caption`) defined in `theme-variables.css`. Mobile typography uses `-mobile` suffix tokens (`text-h2-mobile`, `text-display-mobile`) because these don't map cleanly to other positions on the type scale the way spacing does. Unlike spacing, always use a `-mobile` token for mobile type. Never repurpose a desktop token that happens to share the same pixel value.

| Mobile token | Value | Desktop equivalent |
|--------------|-------|--------------------|
| `text-display-mobile` | 40px | `text-display` (72px) |
| `text-h3-mobile` | 36px | `text-h1` (56px) |
| `text-h2-mobile` | 32px | `text-h2` (48px) |
| `text-h4-mobile` | 24px | `text-h4` (28px) |
| `text-intro-mobile` | 20px | `text-intro` (24px) |
| `text-lg-body-mobile` | 18px | `text-lg-body` (20px) |
| `text-sm-body-mobile` | 14px | `text-sm-body` (16px) |
| `text-caption-mobile` | 12px | `text-caption` (14px) |

## Colors

Colors use semantic names that match Figma. No mapping table needed - `vanilla` in Figma is `bg-vanilla` in code, `plum-700` is `text-plum-700`, etc.

Full palette is in `theme.json` under `settings.color.palette` and mirrored in `theme-variables.css` under `@theme`.

## Border Radius

| Token | Value | Usage |
|-------|-------|-------|
| `rounded` | 4px | Dropdown tiles, small cards |
| `rounded-card` | 16px | Cards (desktop) |
| `rounded-card-mobile` | 8px | Cards (mobile) |
| `rounded-button` | 1000px | Pill buttons |

## Where Things Live

| File | What's in it |
|------|-------------|
| `theme.json` | WP presets - colors, fonts, spacing, radii. Block editor uses these. |
| `assets/theme-variables.css` | Tailwind `@theme` tokens + `:root` semantic aliases. Front-end uses these. |
| `DESIGN-TOKENS.md` | This file. The why behind the what. |
