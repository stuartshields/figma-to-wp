---
name: figma-workflow
description: Project-specific Figma-to-code workflow. Block name gate, component/token maps, spacing translation, structure rules. Use alongside the global /figma skill when implementing from Figma designs.
argument-hint: "[figma URL or block name]"
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Bash, Write, Edit, mcp__figma__get_design_context, mcp__figma__get_screenshot, mcp__figma__get_metadata, mcp__figma__get_variable_defs, mcp__figma__get_code_connect_map
---

# Skill: figma-workflow

## When to Use

Use this skill when implementing UI from a Figma design in your project. This adds project-specific rules on top of the global `/figma` skill. Both should be active during Figma work.

Invoke with `/figma-workflow [URL or block name]`.

## Method

### Step 0: Block Name Gate (Hard Prerequisite)

Before calling ANY Figma MCP tool (`get_design_context`, `get_metadata`, `get_screenshot`, `get_variable_defs`, or anything else), you must know which existing code-side component the Figma design corresponds to: the `wpb/*` block slug, the template part path, or "this is net-new, no existing equivalent."

If the user did not state it in their message, **stop and ask before touching any tool**. Do not guess from the Figma URL. Do not pull `get_metadata` "just to check." Do not pattern-match from a frame name you think you recognise. Designers' Figma naming does not always match the code naming. A Figma component called `Hero / Primary` might be `wpb/primary-hero`, or it might be a brand new variant that needs a new block. Only the user knows.

Ask in this shape:
> Before I pull this from Figma, which of your blocks does it map to? Options I can see in `.claude/figma-component-map.md`: `wpb/primary-hero`, `wpb/secondary-hero`, `wpb/callout-card`, ... (list the plausible candidates). Or is this net-new?

Only after the user answers should you proceed. If the answer is "net-new," confirm the target location (new block in `wp-blocks/blocks/`, new template part in `template-parts/components/`, or something else) before generating code.

### Step 1: Stack Check

Once you know the target component, confirm which stack the work lives in:
- (a) new Gutenberg block (plugin + render file)
- (b) existing block's render file in the theme
- (c) shared template part in `template-parts/components/`
- (d) header/footer modification

Ask the user if still ambiguous.

### Step 2: Read the Maps Before Generating Code

Before calling `get_design_context`, read both:
- `.claude/figma-component-map.md` — block and template part catalogue
- `.claude/figma-token-map.md` — design token translation table

Grep for anything in Figma that looks like an existing entry. **Use existing components, do not invent parallel ones.**

### Step 3: Structure First, Code Second

Call `get_metadata` for the Figma frame before `get_design_context`. List the layer hierarchy and identify which layers map to existing blocks or template parts. Wait for confirmation before generating code.

### Step 4: Prefer `get_design_context` over `get_screenshot`

Screenshots are expensive in tokens and only needed for visual debugging when code output doesn't match intent. Do not fetch screenshots proactively.

### Step 5: Match Figma's Structure

Do not "clean up" nested frames, inline single-child wrappers, or replace divs with semantic tags unless Figma's layer already uses that role. Ugly Figma structure = ugly code structure, by design. The exception is when an entire sub-tree maps to an existing component (e.g., a button). Use the component instead.

### Step 6: Never Hardcode Design Values

Every colour, spacing, font size, radius, shadow, and dimension must come from the token map. If Figma returns a raw hex or pixel value, translate it to the matching named token before writing (for example `#f2796c` becomes `peach`, `12px` becomes `gap-150`). If a Figma value has no matching token, flag it to the user. Do not invent a raw value.

### Mobile vs Desktop Spacing (Critical)

Figma's spacing variables have different values at mobile vs desktop. The project's `theme.json` only holds desktop values because WordPress has no responsive spacing presets. For example, `spacing/150` in Figma may be `12px` on desktop and `8px` on mobile, which translates to `gap-100` (not `gap-150`) when targeting mobile.

See `.claude/figma-token-map.md` "Spacing — Mobile ≠ Desktop" for the per-token reference. The mapping rule below is the single most common source of drift. Do not skip it.

#### Rule

Do NOT create `-mobile` suffix spacing tokens. The existing spacing scale already covers all mobile values at different positions on the scale. Find the matching existing token for the Figma mobile pixel value.

#### How to Apply

1. When implementing templates, check the Figma MCP output's fallback values at each breakpoint (e.g., `var(--spacing/150, 8px)` — the `8px` is the mobile-evaluated value).
2. Find the matching existing `--spacing-*` token for that pixel value at each breakpoint.
3. Use the project's responsive prefix mapping: base = mobile, `sm:` = tablet (640px+), `lg:` = laptop (1024px+), `2xl:` = desktop (1440px+).
4. Only add a prefix when the Figma value changes at that breakpoint. If tablet and laptop share the same value, use `sm:` and skip `lg:`.
5. Example: if a gap is 8px mobile, 12px tablet, 12px laptop, 12px desktop → `gap-100 sm:gap-150` (skip `lg:` and `2xl:` since the value doesn't change after tablet).

#### Desktop ↔ Mobile Mapping Table

| Figma Variable | Desktop Value | Desktop Token | Mobile Value | Mobile Token |
|---|---|---|---|---|
The exact desktop ↔ mobile mapping is project-specific. Build a table like this for your token scale and keep it next to the spacing rule:

| Figma Variable | Desktop Value | Desktop Token | Mobile Value | Mobile Token |
|---|---|---|---|---|
| `spacing/150` | 12px | `*-150` | 8px | `*-100` |
| `spacing/200` | 16px | `*-200` | 12px | `*-150` |
| ... | ... | ... | ... | ... |

This mapping applies to ALL spacing utilities (`gap-`, `p-`, `m-`, `px-`, `py-`, etc.).

Tablet and laptop intermediate values are not in the table — extract them from Figma per-component using the MCP fallback values.

#### Why

WordPress theme.json doesn't support responsive spacing. Creating duplicate `-mobile` tokens would cause token sprawl and diverge from the WP preset system. The community pattern (both WordPress and Tailwind) is to use existing tokens with responsive prefixes in markup.

### Never Reinvent These

When Figma shows something the project already has a shared component for, use the existing component instead of hand-rolling. Maintain a list in your component map of these "use existing" cases. Common categories worth tracking:

- Buttons (CTA pills, with or without icon)
- Icons (inline SVG renderer)
- Expandable body text with a "Read more" toggle
- Responsive images with separate desktop and mobile crops
- Modals (profile bio, lightbox, etc.)
- Breadcrumbs
- Header / nav (modify, do not recreate)
- Footer menu columns

Each of these typically lives in `template-parts/components/` (or wherever your theme keeps shared partials). The component map should record the exact file path so the skill can find it without guessing.

### Keeping the Maps Fresh

The component and token maps are generated by scanning the codebase. Re-run the scan when:

- A new block is added to `wp-blocks/blocks/`
- A block is renamed or removed
- A new shared component lands in `template-parts/components/`
- Design tokens change (`theme.json`, `theme-variables.css`, or your design tokens doc is updated)
- Designers ship a redesign that introduces new Figma component names

Stale maps lead Claude to invent components that already exist, which is the problem the maps are there to prevent.

### MCP Fallback Value Caveat

Don't take Figma MCP fallback values at face value. Spacing variable fallbacks are almost always desktop-mode evaluations, even when pulling a tablet or mobile frame.

1. Always pull the **component definition node**, not the instance node. The component definition resolves variables at the correct mode for that variant.
2. Sanity-check pixel values against the frame width. If 6 tiles with 40px padding each need to fit in 640px and they can't, the values are wrong.
3. Cross-reference the responsive spacing mapping table in the "Mobile vs Desktop Spacing" section above.
4. When the MCP values don't add up, ask the user for the actual values from Figma's inspect panel rather than re-pulling the same node.

### Check All Breakpoints

When touching a block's styles, check ALL breakpoints (mobile, tablet, laptop, desktop) against Figma, not just the breakpoint directly related to the change. After making style changes, fetch the Figma frames for all four breakpoints and compare key properties (positioning, sizing, gradients, padding) against the front-end at each breakpoint. Fix mismatches found, even if they're in code you didn't originally change.

### No Desktop-as-Mobile Image Fallback

Never substitute the desktop image URL as a fallback when the mobile image is missing. Pass the mobile URL as-is (even if empty/false) and let the rendering component decide what to do. When wiring up art-direction-image or any responsive image partial, pass `mobile_url` without `?: $desktop_url` fallbacks. If no mobile image exists, the component renders a plain `<img>` with just the desktop source (which the `<picture>` element handles correctly via its default `<img>`).

## Rules

- **Block name is non-negotiable.** Never call a Figma MCP tool without knowing the target code-side component.
- **Maps before code.** Always read both map files before generating any implementation.
- **Token map is the translation layer.** Raw Figma values (hex, px) must be translated through the token map.
- **Mobile spacing differs from desktop.** The mapping table in the "Mobile vs Desktop Spacing" section above is mandatory reading for every responsive implementation.
- **Reuse existing components.** The "Never reinvent" list is a hard constraint, not a suggestion.
- **MCP values lie about breakpoints.** Pull component definition nodes, not instances. Sanity-check against frame dimensions.
- **Check all four breakpoints.** Not just the one you're changing.
- **No desktop fallback for mobile images.** Pass empty/false, never substitute desktop URL.
