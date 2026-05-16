---
name: block-asset-pipeline
description: Wire a new block's frontend assets (style.scss, view.js) into the legacy webpack + WordPress registration pipeline. Use when adding a non-interactive block that needs CSS or a one-off view script. For new stateful behaviour (toggles, modals, tabs), prefer the Interactivity API skill instead.
argument-hint: "[block-slug]"
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Bash, Write, Edit
---
<!-- Last updated: 2026-05-16T17:00+10:00 -->

# Skill: block-asset-pipeline

## When to Use

Use this skill when creating a new block whose frontend needs CSS (`style.scss`) or a small bespoke `view.js`. For new interactive behaviour (toggles, accordions, modals, carousel state), prefer `/interactivity-api`; the pipeline below is the legacy path that still applies when the Interactivity API doesn't fit (third-party libraries, cross-block coordination, blocks without state).

Editor JS and SCSS are autoloaded; you only need this skill for the **frontend** assets.

## Method

When creating a new block, its frontend assets must be wired into the build and registration pipeline. Missing any step means the block's frontend CSS won't load:

1. **`view.js`**. Create `blocks/<slug>/view.js` that imports `./style.scss`. This is the webpack entry point for frontend assets.
2. **`webpack.config.js`**. Add `'blocks/<slug>/view': './blocks/<slug>/view.js'` to the `entry` object.
3. **`block.json`**. Add both `"style": "wpb-blocks-<slug>-view"` and `"viewScript": "wpb-blocks-<slug>-view"` so WordPress knows which handles to enqueue.
4. **`inc/assets.php`**. Add `register_script_from_build_asset('wpb-blocks-<slug>-view', 'blocks/<slug>/view.js', true);` in `register_block_assets()` so the handle resolves to the built file.
5. **Verify**. After `npm run build`, check that `assets/build/blocks/<slug>/` contains `view.js`, `style-view.css`, and `view.asset.php`.

Editor JS and SCSS are loaded automatically via `autoloadBlocks` in `assets/src/editor.js` (discovers all `blocks/*/index.js` via `require.context`). No manual wiring needed for editor assets.

## Shared Editor Data

A localised global (e.g. `wpbBlocksData`) is set via `wp_localize_script` in `inc/assets.php`. Add new keys there for any theme asset URLs needed in the editor. Document what the global exposes so block editors know what's available.

## Rules

- **Four required wiring steps.** Missing any one means broken frontend CSS.
- **Editor assets are autoloaded.** Don't add editor JS to `inc/assets.php` manually.
- **Prefer Interactivity API for new stateful behaviour.** See `/interactivity-api`. This pipeline is for static CSS and the small set of cases the API doesn't cover.
