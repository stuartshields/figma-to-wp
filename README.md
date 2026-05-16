<!-- Last updated: 2026-05-16T18:00+10:00 -->

# Figma to WP

A starter bundle for using Claude Code to build WordPress blocks from Figma designs. It's the working setup from a real project, packaged so you can drop the parts you want into your own machine and repo.

## What's in here

Two folders that mirror where the files live once installed:

- `project/` goes in your repo's `.claude/` folder. Skills, maps, and project conventions Claude reads on top of your global setup.
- `theme/` is a copy of the design tokens doc that lives inside the WP theme.

A third folder, `project/github-templates/`, holds CI templates you copy into `.github/` for the Playground PR preview workflow.

## What this assumes is already in your `~/.claude/`

This bundle is project-scoped. It expects your global `~/.claude/` already has the agents, rules, and general-purpose skills it leans on. If you don't have them, grab them from your own dotfiles or a separate global bundle.

- **Agents**. `wp` (implementation), `wp-reviewer` (read-only review), `wp-security` (sanitisation), `wp-perf` (queries and assets), `a11y` (WCAG), `ui-review` (usability), `browser-qa-agent` (drives Chrome), `frontend-builder` (parallel component work).
- **Always-on rules**. Communication, discipline, style, dependencies, input-validation. Path-scoped rules like `php-wordpress.md` and `ui-ux.md` are also expected to load when editing matching files.
- **General-purpose skills**. `debug-wp` (structured WP triage), `block-journey` (trace an existing block's files).

The bundle itself does not ship any of these. Install or symlink them in your own `~/.claude/`.

## Compatibility

The patterns in this bundle target the following minimums. Anything newer is documented inline in the skill that uses it, gated behind a feature or version check so older targets still load cleanly.

- **WordPress 6.6+** baseline. Block Bindings (stable since 6.5), Interactivity API (stable since 6.5), and Block API v3 all work on 6.6. WP 6.9+ unlocks the binding dropdown UI (`get_fields_list` field-object shape) and `supports.interactivity: { clientNavigation: true }` for client-side navigation. The `block-bindings` and `interactivity-api` skills gate these behind version checks so 6.6–6.8 sites continue to render.
- **PHP 7.4+**. Examples use typed parameters, return types, and null-coalescing. WordPress core's floor is lower; the bundle's PHP examples assume 7.4 so you don't have to translate.
- **Block API v3** (`"apiVersion": 3` in `block.json`). Required for `viewScriptModule` and the asset pipeline the block-asset-pipeline skill documents.
- **`theme.json` schema v3** (current). The design-tokens pipeline writes to `settings.custom.*`.
- **Tailwind 4** for templates. The CLAUDE.example documents the `bg-(--var)` parentheses syntax this version requires. Don't paste Tailwind 3 bracket syntax.
- **Node 18+** for `@wordpress/scripts` and the Tailwind 4 build pipeline.
- **Figma Dev Mode MCP plugin** for the `mcp__figma__*` tools the figma-workflow skill calls. Install via Figma's plugin browser.
- **Playwright MCP** (optional) for the measurement-driven spec audit in figma-workflow Step 8. Without it the audit falls back to a text comparison and flags visual verification as deferred.

If your target site is on an older WP version, the skills still apply but features marked **6.9+ enhancement** in skill bodies should not be wired up.

## How the project-scoped pieces work

Skills are workflows you trigger with a slash command. Each one is `disable-model-invocation: true`, so Claude only runs them when you ask, not automatically.

- **`/figma-workflow`**. The full Figma-to-code path. Block name gate, frame quality gate, pattern-vs-block branch, MCP plumbing, token translation, mobile spacing rules, measurement-driven spec audit.
- **`/block-dev`**. Entry point for block development. Editor UX, reference-plugin pattern, pointers to the deeper block skills below.
- **`/block-asset-pipeline`**. Wiring `view.js`, webpack, `block.json`, and `inc/assets.php` for a block's frontend CSS and view script.
- **`/block-buttons`**. The `buttons` array attribute, `ButtonGroupEditor`, and the shared `animated-button` render partial.
- **`/block-coding-standards`**. PHP and JS conventions for block code: sanitisation, escaping, docblocks, prettier, DOM contracts.
- **`/block-bindings`**. For new dynamic content surfaces (post meta, taxonomy, custom fields) sourced through stock core blocks. Prefer over a custom `render_callback` block.
- **`/interactivity-api`**. For new block frontend behaviour (toggles, accordions, modals). Prefer over hand-rolled `view.js` + webpack wiring.
- **`/design-tokens`**. When Figma variables change. Re-runs 10up's [`figma-to-wordpress-theme-json-exporter`](https://github.com/10up/figma-to-wordpress-theme-json-exporter), updates `theme.json`, Tailwind aliases in `theme-variables.css`, then refreshes the maps.

Two markdown files sit alongside the skills and get read on every Figma session:

- `figma-component-map.md`: Figma frame → `wpb/*` block (and shared component) lookup.
- `figma-token-map.md`: Figma variable → token lookup. Canonical for the desktop-vs-mobile spacing table.

## Where things go

```
project/CLAUDE.example.md           → <your-repo>/.claude/CLAUDE.md  (rename on copy)
project/figma-*.md                  → <your-repo>/.claude/
project/skills/*                    → <your-repo>/.claude/skills/
project/github-templates/workflows/ → <your-repo>/.github/workflows/
project/github-templates/*.json     → <your-repo>/.github/
theme/DESIGN-TOKENS.md              → <your-wp-theme>/DESIGN-TOKENS.md
```

Copy each block to its destination, then restart Claude Code so it picks up the new files.

## What you need to change before using

`project/CLAUDE.example.md` is a real-world example with project-specific names replaced by placeholders (`wp-blocks`, `wp-theme`, `wpb`). Treat it as a template:

- Rename to `CLAUDE.md`, change the heading, replace each placeholder with your own block plugin name, theme name, and block namespace prefix.
- Drop sections that don't apply. The "no ACF" rule, the top border toggle, and the BEM shared components note are conventions from the source project. Keep them only if they fit yours.
- Keep the shape: stack, where things live, your project conventions, then a pointer to the skills.

The component and token maps are also examples. Regenerate them by scanning your own block plugin and theme tokens, and keep them current. The `design-tokens` skill covers regeneration after Figma changes. Stale maps lead Claude to invent components that already exist, which is the whole problem the maps are there to prevent.

## Typical Figma to WP block flow

You start a session with a Figma URL or a block name and run through something like this:

1. Run `/figma-workflow [URL or block name]`. The skill loads project rules and confirms what you're building.
2. **Block name gate.** It stops and asks which existing block this maps to, before pulling anything from Figma.
3. **Frame quality gate.** It sanity-checks the Figma source (anonymous layers, non-Auto-Layout, instance-only nodes, missing breakpoint variants) before any MCP call.
4. **Pattern or block?** If the frame is a section composed of existing blocks with no new behaviour, route to a pattern PHP file in `<theme>/patterns/` instead of a new `wpb/*` block.
5. **MCP plumbing.** `get_metadata` first if needed, then `get_design_context` on the component definition node (not an instance), declaring which variable mode you're pulling. `get_variable_defs` for tokens. Screenshots only on visual mismatch.
6. **Implementation.** The path-scoped rules (PHP/WordPress, UI/UX) load in the background and shape escaping, accessibility, and visual fixes. The component and token maps stop you reinventing existing components.
7. **Measurement-driven spec audit.** Render the block (locally, or via the Playground PR preview), use Playwright MCP `browser_evaluate` to extract `getComputedStyle` and `getBoundingClientRect`, compare numerically against the MCP values. Visual screenshot diff is only Step 9, and only when the measurements pass but the design still doesn't match.
8. **Review.** Dispatch review agents in parallel: `wp-reviewer`, `a11y`, `ui-review`, plus `wp-security` if you touched input handling.
9. If something misbehaves at runtime, `/debug-wp` walks you through diagnosis.

## Watch your Figma MCP token usage

The Figma MCP runs as a long-lived server that stays connected for the whole session, not a one-off call. Once it's loaded, its tools and instructions sit in your context every turn, and the tools themselves return large payloads. A single `get_screenshot` can run into the thousands of tokens, and a `get_design_context` call on a nested frame can be larger again.

The `/figma-workflow` skill already tries to keep this in check. The rules exist because the cost is real:

- Pull the component definition node, not the instance. Instances drag in parent context, which inflates the response without adding useful information.
- Don't re-pull the same node to double-check. You already have the result; reading it again is wasted tokens.
- Prefer `get_design_context` over `get_screenshot`. Screenshots are only worth fetching when the measurement audit looks fine but the design still doesn't match.
- `get_metadata` first when unsure. Sparse XML, then a targeted `get_design_context` for the few nodes that need code. 5–10× cheaper than deep-fetching the whole frame.
- Skip the figma tools entirely until the block name gate has answered. The gate exists so Claude doesn't burn tokens exploring before it knows what it's building.
- If `get_design_context` truncates on a legitimately large but valid component, raise `MAX_MCP_OUTPUT_TOKENS` in the Claude Code env rather than fanning out into many small calls.

If you find yourself in a long session that has drifted away from Figma work, starting a fresh session for the next task keeps the context clean.

## Example: comparing a Figma frame to an existing block

A useful first move with any new Figma frame is to ask Claude which existing block it maps to, before generating any code. This is the block name gate from `/figma-workflow` in plain prompt form. It stops Claude from inventing a parallel component when one already exists.

A prompt that works:

> Give me a comparison table of this Figma frame against the matching WP block, and tell me the block name. @https://www.figma.com/design/.../node-id=1234-5678

What Claude does with that:

1. Reads `project/figma-component-map.md` to look up candidate blocks.
2. Calls `mcp__figma__get_design_context` once on the node, without fetching a screenshot to keep tokens down.
3. Cross-checks the Figma layer names, tokens, and copy against the candidate block's render template in the theme.
4. Returns the block name and a side-by-side table.

### Sample run

URL: `https://www.figma.com/design/<file-key>/<file-name>?node-id=<node-id>` (a real worked example, redacted)

**Block name:** `wpb/media-split`, configured with `image_position: left`, `background_color: sea-salt`, `type: contained`.

The Status column uses ✅ exact match, ⚙️ configurable via a block attribute, ⚠️ drift from the design, ❌ missing or net-new.

| Aspect | Figma frame ("Image Left Right / Desktop") | `wpb/media-split` block | Status |
|---|---|---|---|
| Layout | Two columns, image left, content right | `image_position: left`, `type: contained` | ⚙️ |
| Page surface | Vanilla `#fffcf5` | Theme page background, section has no bg of its own | ✅ |
| Content panel | Sea-salt `#e7f2ff` | `background_color: sea-salt` (renders `bg-sea-salt` on the right column) | ⚙️ |
| Image radius | Rounded on the left edge only | `rounded-image` on the image's outer corners when bg is set and image is on the left | ⚙️ |
| Content radius | Rounded on the right edge only | `rounded-r-card` on the content panel under the same conditions | ⚙️ |
| Eyebrow | "WHO WE ARE", Neulis Neue Extra Bold 14px, plum-500, 2px tracking, uppercase | `eyebrow` attr, rendered with `font-extrabold text-eyebrow text-plum-500 tracking-[2px] uppercase` | ✅ |
| Heading | "Each day, our team of chefs prepare home-style meals.", Neulis Neue Semi-bold 48px, line-height 52px, plum-700 | `heading` attr, rendered with `text-h2 leading-h2 text-plum-700` | ✅ |
| Body | Neulis Neue Regular 18px, line-height 28px, text-secondary | Inline rich content (`type: contained` with `content` present, not the expandable variant) | ✅ |
| Heading to body gap | 16px (`spacing/200`) | `gap-200` | ✅ |
| Section gap | 40px (`spacing/500`) | `lg:gap-500` | ✅ |
| Content padding | 64px horizontal, 120px vertical | `lg:px-800 lg:py-1500` | ✅ |
| Button | "About who we are", peach bg, plum-700 text, chevron right | `components/animated-button` with `style: solid`, `on_dark: false`, `size: responsive` | ✅ |
| Button radius | 1000px (full pill) | Pill via the shared `animated-button` component | ✅ |

Notes from the comparison:

- This is the with-background variant of the block. The editor user picks `sea-salt` for `background_color` and `left` for `image_position` to reproduce it.
- Because `type` is `contained` and there is content, the body renders inline as a description. The expandable "Read more" version only kicks in when `type` is `expanding`.
- Every visual token in the frame already exists in the theme. Nothing in this design needs a new token.

## Playground PR preview (optional)

`project/github-templates/` contains a GitHub Actions workflow and a Blueprint JSON that boot WordPress Playground on every PR with your plugin + theme loaded and a test page rendering the block under change. Reviewers get a one-click rendered URL; the figma-workflow spec audit (Step 8) gets a deterministic target in CI.

To wire up:

1. Copy `project/github-templates/workflows/playground-preview.yml` to `<your-repo>/.github/workflows/playground-preview.yml`.
2. Copy `project/github-templates/playground-blueprint.json` to `<your-repo>/.github/playground-blueprint.json`.
3. Adjust `plugin-path` and `theme-path` in the workflow to match your layout.
4. Customise the `runPHP` step in the blueprint with the block markup you want to preview. You can replace the `post_content` per branch if needed.

See the [`WordPress/action-wp-playground-pr-preview`](https://github.com/WordPress/action-wp-playground-pr-preview) docs for the full Blueprint surface (seed content, install steps, custom landing pages).

## Notes

The bundle assumes you're already running Claude Code with the official Figma plugin installed (it provides the `mcp__figma__*` tools the skills call). Install that first if it's missing. The measurement-driven spec audit also assumes Playwright MCP is available. It's optional, but without it the audit falls back to a text-comparison check and flags visual verification as deferred.

Rules and skills with a `<!-- Last updated: ... -->` comment are tracked by a staleness check. If you see one older than about a month, review it before relying on it. AI best practices and tooling shift fast.
