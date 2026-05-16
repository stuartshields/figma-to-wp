---
name: block-bindings
description: Use the Block Bindings API for blocks that read dynamic content (post meta, taxonomy terms, custom fields, computed values) instead of routing through a custom render_callback. Stable since WP 6.5, expanded in 6.9.
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Bash, Write, Edit
---
<!-- Last updated: 2026-05-16T14:00+10:00 -->

# Skill: block-bindings

## When to Use

When a block's content is sourced from post meta, taxonomy terms, custom fields, or any per-context dynamic source — and the markup is otherwise a stock core block (paragraph, heading, image, button) or a small variant of one — use the Block Bindings API (stable since WP 6.5, improved in 6.9) instead of building a custom `wpb/*` block with a `render_callback`.

Use this skill **only for new content surfaces**. Existing custom blocks with `render_callback` reads stay as-is; rewrites are a separate decision.

## Why

The legacy pattern is: build a custom block, declare attributes, write a `render_callback` that reads meta/terms, render a template part. That is ~5 files per dynamic content surface.

Block Bindings replaces it with a binding source registered once, then any core block's attributes pointing at the source via `metadata.bindings` in saved block markup. The editor handles the rest.

Result:
- Editors use core blocks they already know.
- One binding source serves many blocks.
- No custom `render_callback` per surface.
- WP 6.9 added `getFieldsList()` so editors see a dropdown of available bindings.

## Method

### Step 1 — Decide if Bindings fit

Suits:
- Author name / bio / avatar on a profile post type
- Recipe scale variants linked from a parent recipe
- Post meta surfaced on the front-end (event date, location, price)
- Custom field values rendered in a heading or paragraph

Doesn't fit:
- Block needs custom editor controls (sidebar toggles, variant pickers)
- Markup structure differs significantly from any core block
- Output requires composing multiple data sources with conditional logic

When in doubt: if you'd reach for a `wpb/*` block with two attributes and a render template, try bindings first.

### Step 2 — Register the binding source (PHP)

```php
namespace WPBlocks\Bindings;

function bootstrap() {
	add_action( 'init', __NAMESPACE__ . '\\register_sources' );
}

function register_sources(): void {
	register_block_bindings_source( 'wpb/post-meta', [
		'label'              => __( 'Post meta', 'wpb' ),
		'get_value_callback' => __NAMESPACE__ . '\\get_post_meta_value',
		'get_fields_list'    => __NAMESPACE__ . '\\get_post_meta_fields',
		'uses_context'       => [ 'postId' ],
	] );
}

function get_post_meta_value( array $source_args, $block_instance ): string {
	$post_id = $block_instance->context['postId'] ?? 0;
	$key = $source_args['key'] ?? '';
	if ( ! $post_id || ! $key ) {
		return '';
	}
	return (string) get_post_meta( $post_id, $key, true );
}

function get_post_meta_fields( $block_instance ): array {
	return [
		'event_date' => __( 'Event date', 'wpb' ),
		'event_location' => __( 'Event location', 'wpb' ),
	];
}
```

`get_fields_list` (WP 6.9+) populates a dropdown for editors. Without it, editors have to type the key by hand.

### Step 3 — Bind from saved block markup

Editors do this via the UI (WP 6.9 binding dropdown), but the saved markup looks like:

```html
<!-- wp:heading {
	"metadata": {
		"bindings": {
			"content": {
				"source": "wpb/post-meta",
				"args": { "key": "event_date" }
			}
		}
	}
} -->
<h2></h2>
<!-- /wp:heading -->
```

The empty `<h2>` is replaced at render time with the bound value.

### Step 4 — Confirm bindable attributes

Not every block attribute is bindable. WP 6.9 expanded the list and added the `block_bindings_allowed_block_attributes` filter to control which attributes can bind. Defaults cover the most common:

| Block | Bindable attributes |
|---|---|
| `core/paragraph` | `content` |
| `core/heading` | `content` |
| `core/button` | `text`, `url`, `linkTarget`, `rel` |
| `core/image` | `id`, `url`, `title`, `alt` |
| `core/image` (6.9+) | `caption` |

If you need to bind an attribute not in the default list, use the filter rather than building a custom block.

## Rules

- **New dynamic content surfaces default to Bindings.** Custom blocks only when the markup differs structurally from any core block.
- **One source per data domain.** Don't create a separate source per field — use `args` to parameterise.
- **`get_fields_list` is not optional.** Editors need a dropdown; WP 6.9 surfaces it automatically.
- **`uses_context` for per-post data.** Without it, the source can't read the current post ID.
- **Escape inside the source callback.** The bound value is interpolated as block content; treat it as user input.
