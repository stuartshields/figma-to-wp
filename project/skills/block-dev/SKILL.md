---
name: block-dev
description: Block development conventions for a WordPress blocks plugin. Editor UX, asset pipeline, buttons, coding standards, reference-plugin patterns. Use when creating or modifying WordPress blocks.
argument-hint: "[block-slug]"
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Bash, Write, Edit
---
<!-- Last updated: 2026-05-16T14:00+10:00 -->

# Skill: block-dev

## When to Use

Use this skill when creating or modifying WordPress blocks in your blocks plugin (typically `content/mu-plugins/wp-blocks/` or `wp-content/mu-plugins/wp-blocks/`). Invoke with `/block-dev [slug]` before starting block work.

## Method

### Reference: existing block plugin

If your project has an established block plugin already (a legacy one, a sibling plugin, or anything that pre-dates this skill), treat it as the reference. **Always check the reference plugin first** for how something is done before building anything new. Match its patterns, components, and conventions. Do not invent new approaches when the reference already has an established way.

Patterns to look for in the reference:
- **Buttons with links:** `RichText` for text + `URLInputButton` for URL (see `components/button/index.js`). Do NOT use `LinkControl` or custom popovers.
- **Conditional rendering:** Check both text and URL before rendering buttons: `if ( empty( $button['text'] ) || empty( $button['url'] ) ) { continue; }`
- **PHP render:** `namespace.php` sanitises attributes → passes to theme template via `Templates\get_extended_template_part()` → template handles output escaping.
- **JSDoc:** `@typedef { import("react").ReactElement } ReactElement` at file top, `@param`/`@return` on components.
- **PHP docblocks:** `@param string[][] $attributes Array of Block attributes.` with blank line before `@return`.

### Editor UX

1. **Inline editing over sidebar:** All primary content (images, text, buttons, links) must be editable directly in the block editor canvas using `RichText`, `MediaUpload`, `MediaReplaceFlow`, `URLInputButton`, etc. Do NOT put content controls in the `InspectorControls` sidebar.
2. **Sidebar is for settings only:** The sidebar (`InspectorControls`) is reserved for non-content settings: style variants, toggles, display options. Not for entering text, URLs, or selecting images.
3. **Always ask:** Before building a new block's editor UI, ask the user: "Should [field X] be inline in the block or in the sidebar?" when the placement isn't obvious. Default preference is always inline, but if the user explicitly requests sidebar placement, respect that.
4. **Style toggles match WP core:** Per-item style selectors (e.g. button fill/outline) should mimic core's block styles panel. Side-by-side preview boxes with labels underneath, not dropdowns or icon-only toggles. See `components/button-style-toggle/index.js`.
5. **Editor sizing uses front-end mobile values:** The editor canvas is narrower than desktop, so editor SCSS should use the front-end's mobile breakpoint values (e.g. `12px 18px` padding, `16px` font) not desktop values.
6. **Pill buttons with RichText + siblings:** When a button has both RichText and extra elements (chevrons, icons), put pill styling (background, border, border-radius, padding) on the wrapper div. The RichText span and siblings render inside. Color goes on the wrapper too so all children inherit via `currentColor`.
7. **Front-end styles leak into the editor.** WordPress loads `style` (from `block.json`) in both the front-end and the editor. Front-end layout constraints (max-width, min-width, fixed heights, media queries) will apply in the editor canvas. Always check if `style.scss` rules need to be reset in `editor.scss`, especially responsive layout properties that don't suit the narrow editor canvas.
8. **Editor preview markup must mirror the front-end render.** The single most common Gutenberg fidelity bug is `edit.js` quietly drifting from the theme template part. Two acceptable patterns:
	- **Shared PHP helper** called from both `render.php` and the editor's `<ServerSideRender>`.
	- **`edit.js` mirrors `render.php` markup exactly** — class names, structure, attributes. Run the figma-workflow spec audit (Step 8) against both surfaces.
	When in doubt, prefer the shared-helper path. Pure client-side `edit.js` reimplementations always drift over time.

### Modern frontend behaviour: Interactivity API

For new blocks needing toggles, accordions, modals, tabs, or carousel state, prefer the WordPress Interactivity API over the legacy `view.js` + webpack entry + `inc/assets.php` pipeline. See `interactivity-api/SKILL.md` for the directive-based pattern. The legacy asset pipeline below remains valid when the API doesn't fit (third-party libraries, cross-block coordination).

### Dynamic content surfaces: Block Bindings

For blocks whose content is sourced from post meta, taxonomy terms, or custom fields and whose markup is a stock core block, prefer the Block Bindings API over a custom `wpb/*` block with a `render_callback`. See `block-bindings/SKILL.md`.

### Block Asset Pipeline

When creating a new block, its frontend assets (style.scss) must be wired into the build and registration pipeline. Missing any step means the block's frontend CSS won't load:

1. **`view.js`** — Create `blocks/<slug>/view.js` that imports `./style.scss`. This is the webpack entry point for frontend assets.
2. **`webpack.config.js`** — Add `'blocks/<slug>/view': './blocks/<slug>/view.js'` to the `entry` object.
3. **`block.json`** — Add both `"style": "wpb-blocks-<slug>-view"` and `"viewScript": "wpb-blocks-<slug>-view"` so WordPress knows which handles to enqueue.
4. **`inc/assets.php`** — Add `register_script_from_build_asset('wpb-blocks-<slug>-view', 'blocks/<slug>/view.js', true);` in `register_block_assets()` so the handle resolves to the built file.
5. **Verify** — After `npm run build`, check that `assets/build/blocks/<slug>/` contains `view.js`, `style-view.css`, and `view.asset.php`.

Editor JS and SCSS are loaded automatically via `autoloadBlocks` in `assets/src/editor.js` (discovers all `blocks/*/index.js` via `require.context`). No manual wiring needed for editor assets.

### Shared Editor Data

- A localized global (e.g. `wpbBlocksData`) is set via `wp_localize_script` in `inc/assets.php`. Add new keys there for any theme asset URLs needed in the editor. Document what the global exposes so block editors know what's available.

### Reusable Components

Live in `wp-blocks/components/{name}/index.js`. Typical examples a project might have:

- `button-link` — RichText + LinkControl popover for inline button editing
- `button-group-editor` — Multi-button editor with inline text + URL popovers, toolbar link button (Cmd+K), configurable max buttons, fill/outline style support
- `button-style-toggle` — WP core-style fill/outline toggle for sidebar

The exact component names and APIs are project-specific. Grep the components folder before building anything new.

### Adding Buttons to a Block

Buttons use the `ButtonGroupEditor` component with a `buttons` **array** attribute (not an object). This is critical. `ButtonGroupEditor` manages the array directly and will break if you convert between object and array.

**block.json:**
```json
"buttons": {
	"type": "array",
	"default": [],
	"items": { "type": "object" }
}
```

**edit.js:**
```js
import ButtonGroupEditor from '../../components/button-group-editor';

const { buttons } = attributes;

<ButtonGroupEditor
	buttons={ buttons }
	setButtons={ ( updated ) => setAttributes( { buttons: updated } ) }
	maxButtons={ 1 }
	isBlockSelected={ isSelected }
	context="light"
	activeButtonIndex={ activeButtonIndex }
	onActiveButtonChange={ setActiveButtonIndex }
/>
```

**namespace.php** (extracting first button for the template):
```php
$raw_buttons = $attributes['buttons'] ?? [];
$button = [];
$first_button = $raw_buttons[0] ?? [];
if ( ! empty( $first_button['text'] ) && ! empty( $first_button['url'] ) ) {
	$button = [
		'text' => sanitize_text_field( $first_button['text'] ),
		'url' => esc_url_raw( $first_button['url'] ),
	];
}
```

**template.php** (rendering with the shared animated-button partial):
```php
<?php if ( ! empty( $button ) ) : ?>
	<?php
	get_extended_template_part( 'components/animated-button', '', [
		'url' => $button['url'],
		'label' => $button['text'],
		'style' => 'solid',
		'on_dark' => false,
	] );
	?>
<?php endif; ?>
```

### Coding Standards

1. **No phpcs:ignore:** Never suppress PHPCS rules. If output triggers an escaping warning, fix it properly. Escape at point of output or construct attributes manually.
2. **No double sanitization, escape at output only:** Don't sanitize text attributes (e.g. `sanitize_text_field()`) in `render_callback` / namespace.php when the template already escapes them at output (`esc_html()`, `wp_kses()`, `esc_attr()`). Block attributes come from the editor's saved post content, not raw user input. The template's output escaping is the real security gate. Pass plain text attributes through raw; escape once in the template. Use `esc_html()` for plain-text fields; reserve `wp_kses()` for fields that intentionally allow specific HTML tags (e.g. headings permitting `<br>` or `<strong>`).
3. **No `get_block_wrapper_attributes()`:** Build wrapper attributes manually. Escape at point of output, not in variable assignment. Pattern:
   ```php
   $wrapper_class = $wrapper_attributes['class'];
   $anchor_id = $attributes['id'] ?? '';
   ?>
   <div class="<?php echo esc_attr( $wrapper_class ); ?>"<?php if ( ! empty( $anchor_id ) ) : ?> id="<?php echo esc_attr( $anchor_id ); ?>"<?php endif; ?>>
   ```
4. **No inline comments that restate the obvious:** Don't add comments like `// Background image z-1` or `/* Hero button sizing, mobile */`. The BEM class names and code structure should be self-documenting.
5. **Comment tone:** Any comments must match the existing human-written style in the reference plugin. Keep them casual and natural. Never use phrasing that sounds AI-generated.
6. **JSDoc on components:** All JS components must have JSDoc matching the reference plugin's pattern: `@typedef { import("react").ReactElement } ReactElement` at file top, and `@param`/`@return` on each exported component.
7. **PHP docblocks:** Match the reference plugin's style: `@param string[][] $attributes Array of Block attributes.` with blank line before `@return`.
8. **No aligned `=>`:** Do NOT align `=>` operators in PHP arrays. Match the reference plugin's style: single space before and after `=>`.
9. **Prettier config:** `.prettierrc.js` extends `@wordpress/prettier-config` with `printWidth: 120`. SCSS uses double quotes. Spaces inside `rgba()`. JS uses WP spacing convention (spaces inside parens/brackets) via `parenSpacing: true`.
10. **Multiline arrays when wide:** PHP arrays with 3+ keys per element, or any array where a single line exceeds ~100 characters, must use one key-value pair per line. Short arrays (1-2 simple keys) can stay on one line.
11. **Use `data-` attributes for JS-PHP DOM contracts:** Front-end JS that reads content from PHP-rendered templates must select elements via `data-` attributes (e.g. `data-profile-name`), never tag selectors, sibling combinators, or style matching. The `data-` attribute is the explicit contract between PHP output and JS behaviour.
12. **Pre-assign defaults, drop the `else`:** When a variable is assigned in every branch of an if/elseif/else, pre-declare it with the most common value before the conditional and drop the final `else`. Only keep branches that override the default.
13. **No em-dashes in inline code comments:** Don't use `—` in `//` or `/* */` comments. Use a period, a comma, or parentheses to break a sentence instead. This rule applies to code comments only, not to markdown docs or user-facing text.

## Rules

- **The reference plugin is the source of truth.** Check it first before building anything new.
- **Inline editing is the default.** Sidebar is for settings only.
- **Asset pipeline has 4 required steps.** Missing any one means broken frontend CSS.
- **Buttons are arrays.** Never convert the `buttons` attribute to/from object format.
- **Escape once at output.** Do not double-sanitize in namespace.php and template.
