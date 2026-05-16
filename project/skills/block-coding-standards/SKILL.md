---
name: block-coding-standards
description: PHP and JS conventions for WordPress block code. Sanitisation, escaping, wrapper attribute construction, comment tone, docblock shape, prettier config, array formatting, DOM-contract patterns. Use when writing or reviewing block PHP / JS in the blocks plugin or the theme render template parts.
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Bash, Write, Edit
---
<!-- Last updated: 2026-05-16T17:00+10:00 -->

# Skill: block-coding-standards

## When to Use

When writing or reviewing PHP / JS inside the blocks plugin (`wp-blocks/`) or in the theme template parts that render blocks (`template-parts/blocks/`, `template-parts/components/`). Invoke before a block code review pass, or before scaffolding a new block from a Figma design.

These are the project's conventions on top of WordPress and `@wordpress/scripts` defaults. The reference plugin is the authoritative example; this skill captures what the reference encodes.

## Standards

1. **No `phpcs:ignore`.** Never suppress PHPCS rules. If output triggers an escaping warning, fix it properly. Escape at point of output or construct attributes manually.
2. **No double sanitisation, escape at output only.** Don't sanitise text attributes (e.g. `sanitize_text_field()`) in `render_callback` / namespace.php when the template already escapes them at output (`esc_html()`, `wp_kses()`, `esc_attr()`). Block attributes come from the editor's saved post content, not raw user input. The template's output escaping is the real security gate. Pass plain text attributes through raw; escape once in the template. Use `esc_html()` for plain-text fields; reserve `wp_kses()` for fields that intentionally allow specific HTML tags (e.g. headings permitting `<br>` or `<strong>`).
3. **No `get_block_wrapper_attributes()`.** Build wrapper attributes manually. Escape at point of output, not in variable assignment. Pattern:
   ```php
   $wrapper_class = $wrapper_attributes['class'];
   $anchor_id = $attributes['id'] ?? '';
   ?>
   <div class="<?php echo esc_attr( $wrapper_class ); ?>"<?php if ( ! empty( $anchor_id ) ) : ?> id="<?php echo esc_attr( $anchor_id ); ?>"<?php endif; ?>>
   ```
4. **No inline comments that restate the obvious.** Don't add comments like `// Background image z-1` or `/* Hero button sizing, mobile */`. The BEM class names and code structure should be self-documenting.
5. **Comment tone.** Any comments must match the existing human-written style in the reference plugin. Keep them casual and natural. Never use phrasing that sounds AI-generated.
6. **JSDoc on components.** All JS components must have JSDoc matching the reference plugin's pattern: `@typedef { import("react").ReactElement } ReactElement` at file top, and `@param`/`@return` on each exported component.
7. **PHP docblocks.** Match the reference plugin's style: `@param string[][] $attributes Array of Block attributes.` with blank line before `@return`.
8. **No aligned `=>`.** Do NOT align `=>` operators in PHP arrays. Match the reference plugin's style: single space before and after `=>`.
9. **Prettier config.** `.prettierrc.js` extends `@wordpress/prettier-config` with `printWidth: 120`. SCSS uses double quotes. Spaces inside `rgba()`. JS uses WP spacing convention (spaces inside parens/brackets) via `parenSpacing: true`.
10. **Multiline arrays when wide.** PHP arrays with 3+ keys per element, or any array where a single line exceeds ~100 characters, must use one key-value pair per line. Short arrays (1-2 simple keys) can stay on one line.
11. **Use `data-` attributes for JS-PHP DOM contracts.** Front-end JS that reads content from PHP-rendered templates must select elements via `data-` attributes (e.g. `data-profile-name`), never tag selectors, sibling combinators, or style matching. The `data-` attribute is the explicit contract between PHP output and JS behaviour.
12. **Pre-assign defaults, drop the `else`.** When a variable is assigned in every branch of an if/elseif/else, pre-declare it with the most common value before the conditional and drop the final `else`. Only keep branches that override the default.
13. **No em-dashes.** See the project-wide writing convention in `CLAUDE.md` (Writing style). This applies in code comments too; use a period, a comma, or parentheses instead.

## Rules

- **Escape once, at output.** Do not double-sanitise in namespace.php and template.
- **No suppressed PHPCS warnings.** Fix the underlying issue.
- **Match the reference plugin.** Style, docblocks, naming. The reference is the authoritative answer when this skill is silent.
