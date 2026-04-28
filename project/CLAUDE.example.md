# Project Conventions (example)

> Replace this header with your project name. The conventions below are a worked example. Adapt them to your stack and remove anything that doesn't apply.

Classic WordPress site: custom block plugin (`wp-blocks`) plus custom theme (`wp-theme`), rendered via Gutenberg. Figma designs come from external designers and are consumed via Dev Mode MCP.

## Stack

- **WordPress** with custom mu-plugin for blocks and custom theme for rendering
- **Gutenberg** block editor (block.json v3, API v3, dynamic rendering)
- **Tailwind 4** for styling (not Tailwind 3 — syntax differs, see Figma section below)
- **PHP** for templates, renders, and server-side logic
- **JS/JSX** for block editor UI only
- **Extended Template Parts** plugin — parameterized `get_extended_template_part()` with `$this->vars[...]` inside templates
- **No ACF** — never use `get_field()`, `the_field()`, or assume ACF is available

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

Every shared component in `template-parts/components/` documents its expected variables in a header docblock. Read the docblock before including a component — do not guess at parameter names.

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

## Figma → Code Workflow

**Full workflow:** `/figma-workflow` skill. Covers block name gate, map files, token translation, mobile spacing, structure rules, and reusable component list.

Key references: `.claude/figma-component-map.md` (block catalogue), `.claude/figma-token-map.md` (token translation).

