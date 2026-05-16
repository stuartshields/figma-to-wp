<!-- TEMPLATE: when you copy this file to .claude/CLAUDE.md, reset the Last updated stamp below to today's date. It's tracked by the staleness rule, and a stale template date pollutes that check. -->
<!-- Last updated: 2026-05-16T17:30+10:00 -->

# Project Conventions (example)

> Replace this header with your project name. The conventions below are a worked example. Adapt them to your stack and remove anything that doesn't apply.

Classic WordPress site: custom block plugin (`wp-blocks`) plus custom theme (`wp-theme`), rendered via Gutenberg. Figma designs come from external designers and are consumed via Dev Mode MCP.

## Stack

- **WordPress 6.6+** baseline. 6.9+ unlocks the Block Bindings dropdown UI (`get_fields_list`) and `supports.interactivity: { clientNavigation: true }` for the Interactivity API router. The `/block-bindings` and `/interactivity-api` skills gate those behind feature checks so 6.6–6.8 sites still load.
- **PHP 7.4+** (typed params, return types, null-coalescing).
- Custom mu-plugin for blocks and custom theme for rendering
- **Gutenberg** block editor (block.json v3, API v3, dynamic rendering)
- **Tailwind 4** for styling (not Tailwind 3; syntax differs, see Figma section below)
- **PHP** for templates, renders, and server-side logic
- **JS/JSX** for block editor UI only
- **Extended Template Parts** plugin: parameterised `get_extended_template_part()` with `$this->vars[...]` inside templates
- **No ACF**. Never use `get_field()`, `the_field()`, or assume ACF is available

## Where things live

- **Blocks (editor UI + registration):** `content/mu-plugins/wp-blocks/blocks/<slug>/`
- **Blocks (frontend render):** `content/themes/wp-theme/template-parts/blocks/<slug>.php`
- **Shared component template parts:** `content/themes/wp-theme/template-parts/components/`
- **Header/footer:** `content/themes/wp-theme/template-parts/header/` and `footer/`
- **Theme tokens:** `content/themes/wp-theme/theme.json` + `assets/theme-variables.css`
- **Design token guide:** `content/themes/wp-theme/DESIGN-TOKENS.md`

Important convention: **blocks are registered in the plugin, but rendered in the theme.** When updating a block's frontend markup, edit the theme's render file, not the plugin's `index.php` or `edit.js` (unless the editor preview also needs updating).

## Tailwind 4 syntax

This theme uses **Tailwind 4**, which has arbitrary CSS variable shorthand:

```html
<!-- Correct -->
<div class="max-w-(--layout-site) bg-(--color-vanilla) h-(--hero-primary-desktop)">

<!-- Wrong (Tailwind 3, do not use) -->
<div class="max-w-[var(--layout-site)] bg-[var(--color-vanilla)]">
```

Preserve parentheses syntax when editing existing templates. Do not convert to bracket syntax.

## Extended Template Parts convention

Use `get_extended_template_part()` (not `get_template_part()`) when a template part needs parameters. Variables are passed via the third argument and read inside the template as `$this->vars[...]`:

```php
get_extended_template_part( 'components/animated-button', '', [
    'url'     => '/contact',
    'label'   => 'Get in touch',
    'style'   => 'solid',
    'on_dark' => false,
    'size'    => 'responsive',
] );
```

Every shared component in `template-parts/components/` documents its expected variables in a header docblock. Read the docblock before including a component. Do not guess at parameter names.

## Block conventions

### Top border toggle

Most blocks support a "Show top border" toggle (`showTopBorder` attribute, default `false`). When enabled it renders `border-t border-melon` with margin-top matching the block's padding-top. The attribute is injected automatically via `inc/top-border.php` for any `wpb/*` block not in the exclude list. The editor toggle is provided by `plugins/top-border-toggle/index.js`.

**When adding a new block** (unless it's a hero, variant, child, or utility block):

1. Pass the attribute in `namespace.php` render_callback: `'show_top_border' => (bool) ( $attributes['showTopBorder'] ?? false ),`
2. Call the helper in the template with margin matching the block's padding-top:
   ```php
   $border_classes = \WPBlocks\Blocks\TopBorder\get_top_border_classes(
       $this->vars['show_top_border'] ?? false,
       'mt-600 sm:mt-750 2xl:mt-1500'
   );
   ```
3. Append `$border_classes` to the wrapper class via `trim()`.
4. If the block should be excluded, add it to `EXCLUDED_BLOCKS` in both `inc/top-border.php` and `plugins/top-border-toggle/index.js`.

### BEM shared components

When shared React components render BEM child elements (e.g. `__items` inside `__profiles`), pass the block's **base class** (e.g. `wp-block-wpb-profile-list`), not an already-suffixed class (e.g. `wp-block-wpb-profile-list__profiles`). Appending `__items` to `__profiles` produces `__profiles__items` which doesn't match the SCSS mixin's `&__items`.

When a shared component needs to generate BEM child classes, accept a `blockClass` prop (the base class) and derive children from it. Check the compiled CSS output matches the DOM classes before shipping.

## Writing style

These rules apply to every surface Claude writes for this project: code, code comments, markdown docs, commit messages, PR descriptions, chat responses, generated UI copy.

- **British English spelling.** colour, behaviour, organise, customise, recognise, analyse, specialise, centre, licence (noun) / license (verb), grey, programme (for a TV/event programme; "program" only for software). WordPress core APIs keep their American spellings (`add_color_palette`, `register_block_pattern_category`); never rename a core function. The rule is about prose, identifiers you create, comments, and UI copy.
- **No em-dashes (long dash).** Use a full stop, a comma, parentheses, or a colon to break a sentence. The long dash reads as AI signature even when otherwise grammatical. Applies in code comments too. Hyphens in compound modifiers (`opt-in`, `read-only`) are fine; en-dashes for numeric ranges (`6.6 to 6.8`, written as words) are fine. The rule targets the long horizontal dash specifically.
- **No emojis** unless the user explicitly asks for them, or you are matching an established pattern in the file you are editing (status icons in tables, etc.).
- **No filler openers** like "Great question,", "Certainly,", "I'd be happy to". Answer the question.
- **Match the tone of the surrounding text.** Code comments in this project are casual and short; PR descriptions are matter-of-fact. Don't write marketing copy in either.

## Skills

- **`/figma-workflow`**. Full Figma-to-code path: block name gate, frame quality gate, pattern-vs-block branch, MCP plumbing, token translation, measurement-driven spec audit. The former global `/figma` skill is folded into this one.
- **`/block-dev`**. Entry point for block development: editor UX, reference-plugin pattern, pointers to the deeper block skills below.
- **`/block-asset-pipeline`**. Wiring `view.js`, webpack, `block.json`, and `inc/assets.php` for a block's frontend CSS and view script.
- **`/block-buttons`**. The `buttons` array attribute, `ButtonGroupEditor`, and the shared `animated-button` render partial.
- **`/block-coding-standards`**. PHP and JS conventions for block code: sanitisation, escaping, docblocks, prettier, DOM contracts.
- **`/block-bindings`**. For new dynamic content surfaces (post meta, taxonomy, custom fields) sourced through stock core blocks. Prefer over a custom `render_callback` block.
- **`/interactivity-api`**. For new block frontend behaviour (toggles, accordions, modals). Prefer over hand-rolled `view.js` + webpack wiring.
- **`/design-tokens`**. When Figma variables change. Re-runs 10up's `figma-to-wordpress-theme-json-exporter`, updates `theme.json`, Tailwind aliases in `theme-variables.css`, then refreshes the maps below.

Key references: `.claude/figma-component-map.md` (block catalogue), `.claude/figma-token-map.md` (token translation).

