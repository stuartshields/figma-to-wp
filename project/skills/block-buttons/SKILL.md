---
name: block-buttons
description: Conventions for adding buttons to a block. Buttons are an array attribute managed by the ButtonGroupEditor component, with text + URL captured via RichText + URLInputButton. Front-end render goes through the shared animated-button template part. Use when a block design includes one or more buttons.
argument-hint: "[block-slug]"
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Bash, Write, Edit
---
<!-- Last updated: 2026-05-16T17:00+10:00 -->

# Skill: block-buttons

## When to Use

When a block design includes one or more buttons, CTAs, or pill-shaped links. The button system is consistent across blocks: an `array` attribute on the block, the `ButtonGroupEditor` component in `edit.js`, and the shared `components/animated-button.php` template part for frontend render. Do not hand-roll buttons.

## Why

Buttons are the single most repeated UI element across the bundle. Centralising the editor controls, the array shape, and the render path means:

- Editors get consistent behaviour (Cmd+K for link, inline text editing, toolbar URL popover) on every block that has a button.
- The visual design (pill shape, hover chevron, solid/transparent variants) lives in one template part. A design tweak in Figma touches one file.
- New blocks copy the same five-line pattern instead of inventing a button shape.

## Method

### Block-side wiring

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

### Components in play

Live in `wp-blocks/components/{name}/index.js`. Typical examples a project might have:

- `button-link`. RichText + LinkControl popover for inline button editing.
- `button-group-editor`. Multi-button editor with inline text + URL popovers, toolbar link button (Cmd+K), configurable max buttons, fill/outline style support.
- `button-style-toggle`. WP core-style fill/outline toggle for sidebar.

The exact component names and APIs are project-specific. Grep the components folder before building anything new.

### Editor patterns specific to buttons

- **Use `RichText` + `URLInputButton`, not `LinkControl` or custom popovers.** Reference: `components/button/index.js` in the existing block plugin.
- **Conditional rendering:** Check both text and URL before rendering a button: `if ( empty( $button['text'] ) || empty( $button['url'] ) ) { continue; }`. Empty entries should not output.
- **Pill buttons with RichText + siblings:** When a button has both RichText and extra elements (chevrons, icons), put pill styling (background, border, border-radius, padding) on the wrapper div. The RichText span and siblings render inside. Colour goes on the wrapper too so all children inherit via `currentColor`.

## Rules

- **`buttons` is an array, never an object.** Never convert between formats; `ButtonGroupEditor` will break.
- **One template part renders all buttons.** `components/animated-button.php`. Do not write a new button partial.
- **Editor uses inline text + URL popovers.** Not `LinkControl`, not a sidebar URL field.
- **Empty entries render nothing.** Guard on both text and URL before emitting markup.
