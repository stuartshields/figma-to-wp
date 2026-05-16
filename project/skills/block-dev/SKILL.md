---
name: block-dev
description: Entry point for block development in a WordPress blocks plugin. Covers the reference-plugin convention and editor UX patterns, and points at dedicated skills for asset wiring, buttons, coding standards, frontend behaviour, and dynamic content. Use when creating or modifying WordPress blocks.
argument-hint: "[block-slug]"
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Bash, Write, Edit
---
<!-- Last updated: 2026-05-16T17:00+10:00 -->

# Skill: block-dev

## When to Use

Use this skill when creating or modifying WordPress blocks in your blocks plugin (typically `content/mu-plugins/wp-blocks/` or `wp-content/mu-plugins/wp-blocks/`). Invoke with `/block-dev [slug]` before starting block work.

This skill is the entry point. It covers the reference-plugin pattern and editor UX. Dedicated skills cover the deeper conventions; see "Related skills" below.

## Method

### Reference: existing block plugin

If your project has an established block plugin already (a legacy one, a sibling plugin, or anything that pre-dates this skill), treat it as the reference. **Always check the reference plugin first** for how something is done before building anything new. Match its patterns, components, and conventions. Do not invent new approaches when the reference already has an established way.

Patterns to look for in the reference:
- **Buttons with links:** `RichText` for text + `URLInputButton` for URL (see `components/button/index.js`). Do NOT use `LinkControl` or custom popovers. See `/block-buttons` for the full convention.
- **Conditional rendering:** Check both text and URL before rendering buttons: `if ( empty( $button['text'] ) || empty( $button['url'] ) ) { continue; }`
- **PHP render:** `namespace.php` sanitises attributes → passes to theme template via `Templates\get_extended_template_part()` → template handles output escaping.
- **JSDoc:** `@typedef { import("react").ReactElement } ReactElement` at file top, `@param`/`@return` on components.
- **PHP docblocks:** `@param string[][] $attributes Array of Block attributes.` with blank line before `@return`.

### Editor UX

1. **Inline editing over sidebar.** All primary content (images, text, buttons, links) must be editable directly in the block editor canvas using `RichText`, `MediaUpload`, `MediaReplaceFlow`, `URLInputButton`, etc. Do NOT put content controls in the `InspectorControls` sidebar.
2. **Sidebar is for settings only.** The sidebar (`InspectorControls`) is reserved for non-content settings: style variants, toggles, display options. Not for entering text, URLs, or selecting images.
3. **Always ask.** Before building a new block's editor UI, ask the user: "Should [field X] be inline in the block or in the sidebar?" when the placement isn't obvious. Default preference is always inline, but if the user explicitly requests sidebar placement, respect that.
4. **Style toggles match WP core.** Per-item style selectors (e.g. button fill/outline) should mimic core's block styles panel. Side-by-side preview boxes with labels underneath, not dropdowns or icon-only toggles. See `components/button-style-toggle/index.js`.
5. **Editor sizing uses front-end mobile values.** The editor canvas is narrower than desktop, so editor SCSS should use the front-end's mobile breakpoint values (e.g. `12px 18px` padding, `16px` font) not desktop values.
6. **Front-end styles leak into the editor.** WordPress loads `style` (from `block.json`) in both the front-end and the editor. Front-end layout constraints (max-width, min-width, fixed heights, media queries) will apply in the editor canvas. Always check if `style.scss` rules need to be reset in `editor.scss`, especially responsive layout properties that don't suit the narrow editor canvas.
7. **Editor preview markup must mirror the front-end render.** The single most common Gutenberg fidelity bug is `edit.js` quietly drifting from the theme template part. Two acceptable patterns:
	- **Shared PHP helper** called from both `render.php` and the editor's `<ServerSideRender>`.
	- **`edit.js` mirrors `render.php` markup exactly**: class names, structure, attributes. Run the figma-workflow spec audit (Step 8) against both surfaces.
	When in doubt, prefer the shared-helper path. Pure client-side `edit.js` reimplementations always drift over time.

## Related skills

This skill stays light on purpose. Depth lives in sibling skills:

- **`/block-asset-pipeline`**. Wiring `view.js`, `webpack.config.js`, `block.json`, and `inc/assets.php` for a block's frontend CSS and view script. Includes the localised editor-data convention.
- **`/block-buttons`**. The `buttons` array attribute, `ButtonGroupEditor`, the `animated-button.php` render partial, and the button-specific editor patterns (pill wrappers, conditional rendering).
- **`/block-coding-standards`**. PHP and JS conventions: sanitisation, escaping, wrapper attributes, docblocks, prettier, DOM contracts.
- **`/interactivity-api`**. Modern frontend behaviour (toggles, accordions, modals, tabs, carousel state) via `data-wp-*` directives. Prefer over the legacy `view.js` + webpack pipeline for new stateful blocks.
- **`/block-bindings`**. Dynamic content surfaces (post meta, taxonomy, custom fields) using stock core blocks. Prefer over a custom `wpb/*` block with a `render_callback`.

## Rules

- **The reference plugin is the source of truth.** Check it first before building anything new.
- **Inline editing is the default.** Sidebar is for settings only.
- **Editor preview mirrors the front-end render.** Shared PHP helper preferred; if you must reimplement in `edit.js`, run the spec audit against both surfaces.
- **Reach for the right sibling skill.** Asset wiring, buttons, standards, interactivity, bindings each have their own skill; do not duplicate that content here.
