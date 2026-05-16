---
name: block-bindings
description: Use the Block Bindings API for blocks that read dynamic content (post meta, taxonomy terms, custom fields, computed values) instead of routing through a custom render_callback. Stable since WP 6.5; baseline patterns here target WP 6.6+. 6.9-only enhancements are flagged inline.
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Bash, Write, Edit
---
<!-- Last updated: 2026-05-16T18:00+10:00 -->

# Skill: block-bindings

## When to Use

When a block's content is sourced from post meta, taxonomy terms, custom fields, or any per-context dynamic source, and the markup is otherwise a stock core block (paragraph, heading, image, button) or a small variant of one, use the Block Bindings API (stable since WP 6.5, improved in 6.9) instead of building a custom `wpb/*` block with a `render_callback`.

Use this skill **only for new content surfaces**. Existing custom blocks with `render_callback` reads stay as-is; rewrites are a separate decision.

## Why

The legacy pattern is: build a custom block, declare attributes, write a `render_callback` that reads meta/terms, render a template part. That is ~5 files per dynamic content surface.

Block Bindings replaces it with a binding source registered once, then any core block's attributes pointing at the source via `metadata.bindings` in saved block markup. The editor handles the rest.

Result:
- Editors use core blocks they already know.
- One binding source serves many blocks.
- No custom `render_callback` per surface.
- On WP 6.9+ the editor surfaces a dropdown of available bindings (see "6.9+ enhancement" below). On 6.6–6.8 editors enter the meta key by hand; the source still resolves at render time.

## Method

### Step 1: Decide if Bindings fit

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

### Step 2: Register the binding source (PHP)

```php
namespace WPBlocks\Bindings;

function bootstrap() {
	add_action( 'init', __NAMESPACE__ . '\\register_sources' );
}

function register_sources(): void {
	$properties = [
		'label'              => __( 'Post meta', 'wpb' ),
		'get_value_callback' => __NAMESPACE__ . '\\get_post_meta_value',
		'uses_context'       => [ 'postId' ],
	];

	// 6.9+ enhancement: dropdown of available fields in the editor.
	// On 6.6–6.8 the key is unknown to register_block_bindings_source(),
	// silently ignored, and editors enter the meta key by hand.
	if ( version_compare( get_bloginfo( 'version' ), '6.9', '>=' ) ) {
		$properties['get_fields_list'] = __NAMESPACE__ . '\\get_post_meta_fields';
	}

	register_block_bindings_source( 'wpb/post-meta', $properties );
}

/**
 * Resolve a bound value for one (block, attribute) pair.
 *
 * The third $attribute_name parameter is what lets one source serve many
 * block attributes. Branch on it when the same source feeds e.g. `content`
 * on a heading and `url` on a button.
 *
 * Escape inside this callback. The return value is interpolated into the
 * block's saved markup at render time, so unescaped meta becomes stored XSS.
 * Pick the escaper by attribute:
 *   - content / text rendered as HTML  → wp_kses_post()
 *   - plain-text attributes             → esc_html()
 *   - URLs                              → esc_url()
 *   - attribute values                  → esc_attr()
 */
function get_post_meta_value( array $source_args, $block_instance, string $attribute_name ): string {
	$post_id = $block_instance->context['postId'] ?? 0;
	$key     = $source_args['key'] ?? '';
	if ( ! $post_id || ! $key ) {
		return '';
	}

	$raw = (string) get_post_meta( $post_id, $key, true );

	// Example targets core/heading + core/paragraph content. Other attributes
	// (url, alt, etc.) need a different escaper. See the table above.
	return wp_kses_post( $raw );
}

/**
 * Return the dropdown options for the WP 6.9+ editor UI.
 *
 * Each entry is a field object with label / type / args. `type` MUST match
 * the bound attribute's declared type or the field won't appear in the
 * dropdown. `core/image.id` is integer, content attributes are string, etc.
 */
function get_post_meta_fields( $block_instance ): array {
	return [
		[
			'label' => __( 'Event date', 'wpb' ),
			'type'  => 'string',
			'args'  => [ 'key' => 'event_date' ],
		],
		[
			'label' => __( 'Event location', 'wpb' ),
			'type'  => 'string',
			'args'  => [ 'key' => 'event_location' ],
		],
	];
}
```

**6.9+ enhancement.** `get_fields_list` populates the binding dropdown in the editor. On 6.6–6.8 the property is unknown and silently ignored. Bindings still resolve at render time, but editors have to type the meta key by hand into the saved block markup. The version gate keeps the registration call working on both.

### Step 3: Bind from saved block markup

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

### Step 4: Confirm bindable attributes

Not every block attribute is bindable. WP 6.9 expanded the list and added the `block_bindings_allowed_block_attributes` filter to control which attributes can bind. Defaults cover the most common:

| Block | Bindable attributes |
|---|---|
| `core/paragraph` | `content` |
| `core/heading` | `content` |
| `core/button` | `text`, `url`, `linkTarget`, `rel` |
| `core/image` | `id`, `url`, `title`, `alt` |
| `core/image` `caption` | 6.9+ only. Falls through to manual content on 6.6–6.8. |

If you need to bind an attribute not in the default list, use the `block_bindings_allowed_block_attributes` filter (6.9+) rather than building a custom block. On 6.6–6.8 the filter is absent; accept the smaller default surface or wait to ship that binding until the site is on 6.9.

## Rules

- **New dynamic content surfaces default to Bindings.** Custom blocks only when the markup differs structurally from any core block.
- **One source per data domain.** Don't create a separate source per field. Use `args` to parameterise and branch on `$attribute_name` when serving multiple block attributes from one source.
- **`get_fields_list` is the 6.9 path, not the floor.** Register it behind a version gate so 6.6–6.8 still loads cleanly; editors on older versions enter the key by hand.
- **`uses_context` for per-post data.** Without it, the source can't read the current post ID.
- **Escape inside the source callback, matched to the bound attribute.** `wp_kses_post()` for HTML-rendering content (`core/heading`, `core/paragraph`), `esc_html()` for plain-text, `esc_url()` for URLs, `esc_attr()` for attribute values. Unescaped meta = stored XSS.
