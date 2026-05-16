---
name: interactivity-api
description: Use WordPress Interactivity API directives for new block frontend behaviour instead of hand-rolled view.js + webpack + inc/assets.php wiring. Stable since WP 6.5; baseline patterns here target WP 6.6+. 6.9-only enhancements (client navigation object form, expanded router) are flagged inline.
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Bash, Write, Edit
---
<!-- Last updated: 2026-05-16T17:30+10:00 -->

# Skill: interactivity-api

## When to Use

When a new block needs frontend behaviour (toggles, accordions, carousels, modals, search filters, form validation), prefer the Interactivity API over hand-rolled `view.js` + webpack entry + `inc/assets.php` wiring.

Use this skill **only for new blocks or new behaviour**. Existing blocks on the legacy `view.js` path stay there; rewrites are a separate decision.

Do NOT use for editor-side behaviour (that's `edit.js`).

## Why

The legacy path documented in `block-asset-pipeline/SKILL.md` requires 4 wiring steps per block (`view.js` → webpack entry → `block.json` `viewScript` → `inc/assets.php` register). The Interactivity API replaces it with directives in the rendered HTML and one `store()` call.

Result:
- Less bundler glue per block.
- SSR-friendly hydration (state matches server render).
- Declarative state in markup, not imperative DOM queries.
- Smaller block source, lower agent token cost on edits.

## Method

### Step 1: Decide if Interactivity API fits

Suits:
- Toggle/expand UI (accordions, "Read more")
- Tabs / tab panels
- Modals with open/close state
- Simple sliders / carousels with index state
- Filter / search UIs with reactive lists

Doesn't fit:
- Complex third-party JS with its own DOM ownership (heavy carousels, charts)
- DOM manipulation outside the block's own subtree
- Anything reading state from outside the block tree

### Step 2: Mark the block as interactive in `block.json`

Baseline (works on 6.6+):

```json
{
	"supports": {
		"interactivity": true
	},
	"viewScriptModule": "file:./view.js"
}
```

`viewScriptModule` (not `viewScript`) tells WordPress to load this as an ES module. The Interactivity API requires modules.

**6.9+ enhancement, client-side navigation.** If the block needs to participate in `@wordpress/interactivity-router` client navigation (state persists across page transitions, no full reload), declare the object form on WP 6.9+:

```json
{
	"supports": {
		"interactivity": { "clientNavigation": true }
	},
	"viewScriptModule": "file:./view.js"
}
```

The object form auto-marks the `viewScriptModule` as navigation-compatible. On 6.6–6.8 WordPress doesn't read the nested key, so the block still loads but won't participate in router navigation. If a site needs to support both, keep the baseline `interactivity: true` and only flip to the object form once 6.9 is the floor.

### Step 3: Add directives to the rendered HTML

In `render.php` or the theme template part, add `data-wp-*` directives:

```php
<div
	<?php echo wp_interactivity_data_wp_context( [ 'isOpen' => false ] ); ?>
	data-wp-interactive="wpb/faq-item"
>
	<button
		data-wp-on--click="actions.toggle"
		data-wp-bind--aria-expanded="context.isOpen"
	>
		<?php echo esc_html( $question ); ?>
	</button>
	<div data-wp-bind--hidden="!context.isOpen">
		<?php echo wp_kses_post( $answer ); ?>
	</div>
</div>
```

Key directives:
- `data-wp-interactive="namespace"`: declares the store namespace for this subtree
- `data-wp-context`: initial state (use `wp_interactivity_data_wp_context()` helper for safe JSON)
- `data-wp-on--<event>`: event handler
- `data-wp-bind--<attr>`: reactive attribute binding
- `data-wp-class--<class>`: conditional class
- `data-wp-effect`: side effects when state changes

### Step 4: Register the store in `view.js`

```js
import { store, getContext } from '@wordpress/interactivity';

store( 'wpb/faq-item', {
	actions: {
		toggle: () => {
			const ctx = getContext();
			ctx.isOpen = ! ctx.isOpen;
		},
	},
} );
```

### Step 5: Build wiring

`@wordpress/scripts` handles the module build automatically when `viewScriptModule` is set in `block.json`. No webpack entry to add manually. No `inc/assets.php` registration to write. WordPress resolves the module from `block.json`.

Verify after `npm run build` that the module lands in `assets/build/blocks/<slug>/view.js`.

### Client-side navigation (optional, 6.9 expanded)

`@wordpress/interactivity-router` lets a block opt into SPA-like navigation: clicking a link inside a router-aware region fetches the new page, swaps the DOM, and preserves the interactive state declared in `data-wp-context`. The router has been available since 6.5 but covers more cases in 6.9 (better stylesheet/module reuse, support for `data-wp-on-document` listeners across navigations).

Two integration points:

- **Block opts in** via `supports.interactivity: { clientNavigation: true }` (6.9+, see Step 2 above) or by calling `add_client_navigation_support_to_script_module()` from PHP for manually-registered modules.
- **Link declares intent** in markup: `<a data-wp-on--click="actions.navigate" href="...">` with a store action that calls `actions.navigate({ url })` from `@wordpress/interactivity-router`.

Don't ship router navigation on a 6.6 site. Pages still load, but reused-stylesheet behaviour and a few directive lifecycles only work on 6.9+. The skill defaults to non-router for that reason.

### Editor preview parity

The Interactivity API does not run in the editor. For editor preview of stateful behaviour:

- Show the **default state** (whatever the initial `context` value is) in `edit.js`.
- Document the interactive behaviour in a help panel if non-obvious.
- Don't try to reimplement the store in React. It drifts.

## Falling back to the legacy path

Use the legacy `view.js` + `inc/assets.php` path when:

- The behaviour requires a third-party library with its own DOM ownership.
- You need cross-block state coordination (one block tells another to scroll).
- The block is being added to a part of the codebase that doesn't yet have `@wordpress/scripts` interactivity module support configured.

The legacy path is documented in `block-asset-pipeline/SKILL.md`.

## Rules

- **New blocks default to Interactivity API.** Legacy path only when the API doesn't fit.
- **`viewScriptModule`, not `viewScript`.** The Interactivity API requires ES modules.
- **Baseline `supports.interactivity: true` is the safe floor.** Only switch to the `{ clientNavigation: true }` object form once 6.9 is the minimum supported version.
- **`data-wp-context` for initial state.** Use the `wp_interactivity_data_wp_context()` PHP helper to emit safe JSON.
- **Namespace per block.** `wpb/<slug>` matches the block name and avoids store collisions across the page.
- **No state in DOM attributes.** Use the store. The store's reactivity is the whole point.
- **Editor preview shows default state only.** Don't reimplement the store in React.
