---
name: design-tokens
description: Keep Figma variables, theme.json, and Tailwind @theme tokens in sync. Uses 10up's figma-to-wordpress-theme-json-exporter as the canonical export path; theme.json is the source of truth and Tailwind aliases via --wp--custom--* CSS variables.
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Bash, Write, Edit
---
<!-- Last updated: 2026-05-16T14:00+10:00 -->

# Skill: design-tokens

## When to Use

When design tokens change in Figma — a new colour, a renamed spacing variable, a new typography size — and the change needs to flow into `theme.json` (which drives the block editor and Global Styles) and `theme-variables.css` (which drives Tailwind utility classes in templates).

Invoke before re-running the token map regeneration.

## Source of Truth

**Direction: theme.json is canonical for design tokens.**

1. **Figma variables** are authored by designers.
2. **`theme.json`** is generated from Figma using 10up's [figma-to-wordpress-theme-json-exporter](https://github.com/10up/figma-to-wordpress-theme-json-exporter) plugin. Everything project-specific lands under `settings.custom.*`, emitting `--wp--custom--<group>--<name>` CSS variables.
3. **`assets/theme-variables.css`** is a thin `@theme { ... }` block that aliases `--wp--custom--*` to short Tailwind-friendly names (`--color-peach`, `--spacing-150`, etc.).
4. **`figma-token-map.md`** is the lookup table the figma-workflow skill reads on every Figma session.

This is a one-way pipeline: Figma → `theme.json` → Tailwind aliases. Never hand-edit `theme.json` for design tokens — re-run the exporter.

Component-specific dimensions (hero heights, modal widths, carousel offsets) the exporter does not emit are hand-authored in `theme-variables.css` under `:root` and documented in `DESIGN-TOKENS.md`.

## Method

### Step 1 — Export from Figma

Run 10up's plugin in Figma Dev Mode (designer or developer):

1. Open the Figma file in Dev Mode.
2. Run the **Figma to WordPress theme.json Exporter** plugin.
3. Choose the export scope (all variables, or a single collection).
4. Configure Desktop / Mobile mode pairs if any tokens need fluid min/max values.
5. Export. The plugin emits a `theme.json` fragment plus optional style-variation files.

Save the output. The next steps run in this repo.

### Step 2 — Merge into the project's `theme.json`

The exporter emits a `settings.custom` block. Merge into `<theme>/theme.json`:

- Tokens nest under `settings.custom.color`, `settings.custom.spacing`, `settings.custom.typography`, etc.
- Each token emits a CSS variable named `--wp--custom--<group>--<name>` (kebab-case).
- Don't overwrite the rest of `theme.json` — only the `settings.custom` subtree.

Project-specific tokens the exporter doesn't reach stay in `settings.custom` too, hand-authored.

### Step 3 — Update the Tailwind alias file

`<theme>/assets/theme-variables.css` is a Tailwind v4 `@theme` block aliasing WP custom properties to short names:

```css
@theme {
	--color-peach: var(--wp--custom--color--peach);
	--color-plum-700: var(--wp--custom--color--plum-700);
	--spacing-150: var(--wp--custom--spacing--150);
	/* etc. */
}
```

Add an alias for each new token. Keep alias names matching the existing scale (`gap-150`, `bg-peach`, `text-h2`).

For component-specific dimensions the exporter doesn't emit (e.g. `--hero-primary-desktop`), author the CSS variable directly in the same file under `:root` and document in `DESIGN-TOKENS.md`.

### Step 4 — Refresh `figma-token-map.md`

After `theme.json` + `theme-variables.css` are updated:

1. Read `theme.json` `settings.custom` for all colour/spacing/typography tokens.
2. Read `theme-variables.css` for any CSS-only tokens not in `theme.json`.
3. Regenerate `figma-token-map.md` so the Figma-name → token columns match.

Don't hand-author the map — it gets stale. Update the `<!-- Last updated -->` comment when regenerated.

### Step 5 — Refresh `DESIGN-TOKENS.md`

`theme/DESIGN-TOKENS.md` documents the *why* — token philosophy, the mobile-vs-desktop spacing rule, where things live. Update narrative sections only if the rules changed. Don't duplicate the lookup table; reference `figma-token-map.md` instead.

## When to Re-Run

- Designer adds, renames, or removes a Figma variable.
- A new breakpoint mode is added in Figma.
- A token's value changes in Figma.
- A new component-specific dimension is needed in the theme (hand-author in this case — the exporter doesn't reach it).

## Rules

- **`theme.json` is canonical for design tokens.** Don't hand-edit it for design tokens — re-run the exporter.
- **Tailwind aliases live in one file** (`assets/theme-variables.css`) and reference `--wp--custom--*`.
- **Component-specific dimensions** are hand-authored in `theme-variables.css`, not generated.
- **The maps refresh after the source files.** Never the other way round.
- **One CSS custom property per token, never duplicated.** Don't define the same value in both `theme.json` and `theme-variables.css` directly — alias.
