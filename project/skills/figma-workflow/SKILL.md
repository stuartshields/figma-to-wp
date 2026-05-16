---
name: figma-workflow
description: Figma-to-code workflow for WordPress blocks. MCP plumbing, block name gate, frame quality gate, component/token maps, pattern-vs-block branch, measurement-driven spec audit, mobile spacing. Folds the former global `/figma` MCP skill into a single project-scoped workflow.
argument-hint: "[figma URL or block name]"
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Bash, Write, Edit, mcp__figma__get_design_context, mcp__figma__get_screenshot, mcp__figma__get_metadata, mcp__figma__get_variable_defs, mcp__playwright__browser_navigate, mcp__playwright__browser_take_screenshot, mcp__playwright__browser_snapshot, mcp__playwright__browser_resize, mcp__playwright__browser_evaluate
---
<!-- Last updated: 2026-05-16T17:30+10:00 -->

# Skill: figma-workflow

## When to Use

Use this skill when implementing UI from a Figma design in this project. It covers the full path: gating the request, pulling design context from the Figma Dev Mode MCP, translating tokens, writing code, and verifying parity against the Figma source.

Invoke with `/figma-workflow [URL or block name]`. Use `$ARGUMENTS` as the URL or name; if empty, ask.

Do NOT use this skill for general frontend work that does not involve a Figma design.

## Method

The order matters. Each step is a gate; don't skip ahead.

### Step 0: Block Name Gate (hard prerequisite)

Before calling any Figma MCP tool, you must know which existing code-side component the Figma design corresponds to: a `wpb/*` block slug, a template part path, a pattern, or "net-new."

If the user did not state it, stop and ask:

> Before I pull this from Figma, which of your blocks does it map to? Options I can see in `.claude/figma-component-map.md`: `wpb/...`, `wpb/...`. Or is this net-new?

Don't guess from the Figma URL. Don't pull `get_metadata` to peek. Don't pattern-match from a frame name. Designers' Figma naming does not always match code naming.

If "net-new," confirm the target location before generating code (new block in `wp-blocks/blocks/`, new template part in `template-parts/components/`, new pattern in `<theme>/patterns/`, or something else).

### Step 0a: Frame Quality Gate

After the block name is set, sanity-check the Figma source. Reject any of the following before touching MCP:

- **Anonymous layer names** in the target subtree (`Rectangle 1023`, `Frame 42`, `Group 17`). Anonymous layers produce garbage output. There is nothing semantic to name elements by.
- **Non-Auto-Layout root frame.** Anything not in Auto Layout end-to-end produces absolute-positioning code. Ask the designer to convert to Auto Layout first.
- **Instance-only target node.** `get_design_context` on an instance yields override-only data and misses the main component's props. If the user passed an instance URL, ask for the main component URL.
- **Missing breakpoint variants** when a responsive block is in scope. If the Figma only has desktop, ask for the mobile/tablet variants before implementing.

If any of these fail, explain what's wrong and ask the user to fix the Figma source or confirm proceeding with reduced fidelity expectations.

### Step 1: Pattern or Block?

If the Figma frame is a section composed of existing blocks with no new behaviour and no new attributes (just a particular arrangement), it is a **pattern**, not a new block.

| Signal | Likely outcome |
|---|---|
| New visual primitive, new attributes, new editor controls | new `wpb/*` block |
| Existing blocks arranged in a particular layout, no new attributes | `register_block_pattern()` in `<theme>/patterns/` |
| One existing block with a new visual variant only | `styles` array in the block's `block.json` + a SCSS rule |
| Dynamic content surface (post meta, taxonomy, custom field) using stock blocks | Block Bindings: see `block-bindings/SKILL.md` |

If pattern: skip the asset pipeline. Write a pattern PHP file in `<theme>/patterns/` with a docblock header (`Title`, `Slug`, `Categories`, `Block Types`, `Description`), and the content as static block markup. The token translation and spec audit steps below still apply.

### Step 2: Stack Check

Confirm which stack the work lives in:

- (a) new Gutenberg block (plugin + theme render file)
- (b) existing block's render file in the theme
- (c) shared template part in `template-parts/components/`
- (d) header/footer modification
- (e) pattern PHP file in `<theme>/patterns/`

Ask if still ambiguous.

### Step 3: Read the Maps

Before any `get_design_context`, read both:

- `.claude/figma-component-map.md`: block and template part catalogue
- `.claude/figma-token-map.md`: design token translation table

Grep for anything in Figma that looks like an existing entry. Use existing components; do not invent parallel ones.

### Step 4: MCP Plumbing (call order matters)

MCP tools are mandatory. Never implement from memory.

**Call order:**

1. **`get_metadata`** first if the target is a large frame or component set. Returns a sparse XML outline so you can pick the right child nodes. Skip when the target is a single small node.
2. **`get_design_context`** on the **component definition node**, not on an instance. Instances drag in parent context, inflate the response, and miss the main component's props. If you only have an instance URL, ask for the definition or use `get_metadata` to navigate to it.
3. **Declare the variable mode** when the design has separate mobile/tablet/desktop variable modes. Pulling a mobile node without specifying the mobile mode returns desktop values silently. State the mode you're pulling at the top of your reasoning so the user can correct you if wrong.
4. **`get_variable_defs`** when the design uses Figma variables/tokens. Map to the token system, never hardcode.
5. **`get_screenshot`** only when the implementation diverges from the design and you need a visual reference to debug (Step 9). Not for initial implementation.

**If any tool call fails or returns incomplete data:** tell the user. Do not fill gaps from assumptions.

**If `get_design_context` truncates** on a legitimately large but valid component, raise `MAX_MCP_OUTPUT_TOKENS` in the Claude Code env rather than fanning out into many small calls. Most truncations mean the wrong node was pulled. `get_metadata` first.

### Step 5: Responsive Variants

If the design has separate mobile/tablet/desktop variants (variable modes, frame siblings, or component variants):

1. **Identify all breakpoint variants before writing code.** Use `get_metadata` on the parent if needed.
2. **Fetch `get_design_context` for every breakpoint** with the matching variable mode declared. Don't fetch desktop and guess mobile.
3. **Build a comparison table** of values that differ vs values that stay the same. This is the responsive spec.

This gate is blocking. Do not write responsive CSS without MCP data for every breakpoint the design targets.

### Step 6: Match Figma's Structure

Compare DOM structure against Figma's layer hierarchy. Do not "clean up" nested frames, inline single-child wrappers, or replace `div` with semantic tags unless Figma's layer already uses that role. Ugly Figma structure = ugly code structure, by design.

Exception: when an entire sub-tree maps to an existing component (button, icon, expandable body, art-direction-image, modal, etc.), use the component instead; see `figma-component-map.md`.

### Step 7: Never Hardcode Design Values

Every colour, spacing, font size, radius, shadow, and dimension must come from `figma-token-map.md`. If Figma returns a raw hex or pixel value, translate to the matching named token (`#f2796c` → `peach`, `12px` → `gap-150`). If a Figma value has no matching token, flag it to the user. Don't invent a raw value.

For the desktop ↔ mobile spacing translation rule, see `figma-token-map.md`. That table is the single source of truth. Do not duplicate it here.

### Step 8: Measurement-Driven Spec Audit

After writing the CSS/SCSS, **measure the rendered output and compare to the MCP data**. This is a numerical check, not a visual one. Visual diffs are unreliable due to font hinting and anti-aliasing.

1. **Render the block in a deterministic environment.** Local dev server (`wp-now`, the project's `npm run start`), or the Playground PR preview if configured (see `project/github-templates/`).
2. **Extract computed styles** via `mcp__playwright__browser_evaluate`. For each audited element, return measured values:

	```js
	const el = document.querySelector('.target');
	const cs = getComputedStyle(el);
	const rect = el.getBoundingClientRect();
	return {
		fontSize: cs.fontSize,
		lineHeight: cs.lineHeight,
		paddingTop: cs.paddingTop,
		paddingRight: cs.paddingRight,
		gap: cs.gap,
		borderRadius: cs.borderRadius,
		width: rect.width,
		height: rect.height,
	};
	```

3. **Compare numerically against MCP values** from `get_design_context`. Present a verification table:

	```
	| Element  | Property    | Figma (desktop) | Measured | Match?     |
	|----------|-------------|-----------------|----------|------------|
	| .heading | font-size   | 48px            | 48px     | ✅         |
	| .content | padding-top | 64px            | 60px     | ❌ (-4px)  |
	```

4. **Any non-match is a bug.** Fix and re-measure. Do not skip a row because "it looks right visually."
5. **Repeat at every breakpoint** the design targets. Resize via `browser_resize` between rows.

**Local fallback:** if no rendered URL is available, fall back to re-reading the CSS and comparing token names against MCP output. Flag visual verification as deferred in your summary so the user knows it hasn't actually been measured.

### Step 9: Visual Verification (only on mismatch)

Screenshots are only worth fetching when the measured audit looks fine but the design still doesn't match, i.e. there's a layout or composition bug the numbers don't catch.

- Call `get_screenshot` from Figma for the reference image.
- Take a browser screenshot via `mcp__playwright__browser_take_screenshot`.
- Compare side-by-side. Iterate.

**Visual iteration limit: 3 rounds.** After 3 compare-fix-screenshot cycles without parity, stop. Report what matches, what doesn't, and what you tried. Remaining deltas may need designer input or are platform-specific rendering differences.

## Supporting Notes

### Never Reinvent

When Figma shows something the project already has a shared component for, use the existing component. See `figma-component-map.md` for the canonical list (buttons, icons, expandable text, art-directed responsive images, modals, breadcrumbs, header/nav, footer). The exact filenames and APIs live there. Do not duplicate them in this skill.

### Editor Preview Parity

When building a new block, the editor preview (`edit.js`) and the front-end render (`render.php` / theme template part) must produce equivalent markup. Two acceptable patterns:

1. **Shared PHP helper** called from both `render.php` and the editor's `<ServerSideRender>`.
2. **`edit.js` mirrors `render.php` markup exactly**: class names, structure, attributes. Run Step 8 against both surfaces.

The single most common Gutenberg fidelity bug is `edit.js` quietly drifting from the front-end render. The spec audit catches it if you actually run it on both surfaces. See `block-asset-pipeline/SKILL.md` for the wiring and `block-dev/SKILL.md` for the editor-vs-frontend CSS reset rules.

### Keeping the Maps Fresh

The component and token maps are generated by scanning the codebase. Re-run the scan when:

- A new block is added to `wp-blocks/blocks/`
- A block is renamed or removed
- A new shared component lands in `template-parts/components/`
- Design tokens change (`theme.json`, `theme-variables.css`, or `DESIGN-TOKENS.md` is updated; see `design-tokens/SKILL.md`)
- Designers ship a redesign that introduces new Figma component names

Stale maps lead Claude to invent components that already exist. The problem the maps exist to prevent.

### Check All Breakpoints

When touching a block's styles, check all four breakpoints (mobile, tablet, laptop, desktop) against Figma, not just the one related to the change. After making style changes, run Step 5 (responsive variants) and Step 8 (measurement audit) at each breakpoint. Fix mismatches even when they're in code you didn't originally change.

### No Desktop-as-Mobile Image Fallback

Never substitute the desktop image URL as a fallback when the mobile image is missing. Pass the mobile URL as-is (even if empty/false) and let the rendering component decide what to do. When wiring `art-direction-image` or any responsive image partial, pass `mobile_url` without `?: $desktop_url` fallbacks.

### Token Hygiene (long Figma sessions)

The Figma MCP runs as a long-lived server. Tools and instructions stay in context every turn, and tool responses are large. Mitigations:

- **Call by deep node id, not by page.** Pasting a deep Figma URL with `node-id=...` cuts most of the cost.
- **`get_metadata` first when unsure.** Sparse XML, then a targeted `get_design_context` for the few nodes that need code. 5–10× cheaper than deep-fetch first.
- **Don't re-pull the same node** to double-check. You already have the result.
- **Skip `get_screenshot`** unless you're in Step 9 (visual verification on mismatch).
- **Cache the initial Figma screenshot** if you needed one. Don't re-fetch on every iteration cycle.
- **`MAX_MCP_OUTPUT_TOKENS`** raises the truncation ceiling. Use sparingly, it also raises cost.
- **Long session drifting away from Figma work?** Start a fresh session for the next task to keep context clean.

## Rules

### Gates (Steps 0–1)

- **Block name is non-negotiable.** Never call a Figma MCP tool without knowing the target code-side component.
- **Frame quality is non-negotiable.** Anonymous layers, non-Auto-Layout roots, instance-only nodes, and missing breakpoint variants are gates, not warnings.

### MCP (Steps 3–5)

- **Maps before code.** Always read both map files before generating implementation.
- **Component definition nodes, not instances.** Pull the main component, not an instance, for `get_design_context`.
- **Variable mode declared.** When the design uses mobile/tablet/desktop modes, state which mode you're pulling.

### Implementation (Steps 6–7)

- **Token map is the translation layer.** Raw Figma values (hex, px) translate through `figma-token-map.md`.
- **Reuse existing components.** The "Never reinvent" list in the component map is a hard constraint, not a suggestion.
- **MCP output is a reference, not final code.** Translate to project conventions, framework, and existing components.

### Audit (Steps 8–9)

- **Measure, don't squint.** The spec audit is `getComputedStyle` against MCP values. Text comparison is the fallback when no rendered URL exists.
- **Check all four breakpoints.** Not just the one you're changing.
- **Screenshots on mismatch only.** Visual verification is Step 9, not Step 1.
- **3-round iteration limit on visual fixes.** Stop and report after 3 cycles without parity.
- **If the user says it doesn't look right, they're right.** Don't explain why the CSS "should" work. Look at what's rendering.
