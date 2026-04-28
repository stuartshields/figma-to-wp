# Figma to WP

A starter bundle for using Claude Code to build WordPress blocks from Figma designs. It's the working setup from a real project, packaged so you can drop the parts you want into your own machine and repo.

## What's in here

Three folders that mirror where the files live once installed:

- `global/` goes in `~/.claude/`. Rules, skills, and agents that apply to every project on your machine.
- `project/` goes in your repo's `.claude/` folder. The project-specific layer Claude reads on top of the global one.
- `theme/` is a copy of the design tokens doc that lives inside the WP theme.

## How the pieces work

There are three kinds of things Claude Code uses, and they fire at different times.

Rules are short markdown files Claude reads automatically. Some load every session (the always-on baseline like `communication.md`, `discipline.md`, `style.md`). Others have a `paths:` glob in the frontmatter and only load when you're editing matching files. For example, `php-wordpress.md` only fires for `*.php`, and `ui-ux.md` only fires for templates, themes, and blocks. You don't invoke rules, they just shape how Claude works.

Skills are workflows you trigger with a slash command. The Figma to code flow lives in skills: `/figma` handles the MCP plumbing, `/figma-workflow` adds the project rules on top, `/block-dev` covers block conventions, `/block-journey` traces all the files for an existing block, and `/debug-wp` runs a structured interview when something breaks. Skills don't run unless you ask for them, but once you do, Claude follows their steps.

Agents are subagents you dispatch for a single job. They run in their own context window and report back, which is useful when you want isolated work or want to run a few in parallel. The bundle ships the WordPress and UI ones: `wp` for implementation, `wp-reviewer` for read-only code review, `wp-security` for sanitisation checks, `wp-perf` for query and asset audits, `ui-review` for usability, `a11y` for WCAG, `browser-qa-agent` for driving Chrome on a running site, and `frontend-builder` for parallel component work.

## Where things go

```
global/rules/*           →  ~/.claude/rules/
global/skills/*          →  ~/.claude/skills/
global/agents/*          →  ~/.claude/agents/
project/CLAUDE.example.md → <your-repo>/.claude/CLAUDE.md  (rename on copy)
project/skills/*         →  <your-repo>/.claude/skills/
project/figma-*.md       →  <your-repo>/.claude/
theme/DESIGN-TOKENS.md   →  <your-wp-theme>/DESIGN-TOKENS.md
```

Copy each block to its destination, then restart Claude Code so it picks up the new files.

## Typical Figma to WP block flow

You start a session with a Figma URL or a block name and run through something like this:

1. Run `/figma-workflow [URL or block name]`. The skill loads the project rules and confirms what you're building.
2. It stops at a block name gate and asks which existing block this maps to, before pulling anything from Figma. You answer, then it continues.
3. It reads the component and token maps, fetches design context through the global `/figma` skill, and walks through structure before generating code.
4. As you write the implementation, the path-scoped rules (`php-wordpress.md`, `ui-ux.md`) load in the background and shape escaping, accessibility, and visual fixes.
5. After the build, dispatch review agents in parallel: `wp-reviewer`, `a11y`, `ui-review`, plus `wp-security` if you touched input handling.
6. If something misbehaves at runtime, `/debug-wp` walks you through diagnosis.

## What you need to change before using

`project/CLAUDE.example.md` is a real-world example with project-specific names replaced by placeholders (`wp-blocks`, `wp-theme`, `wpb`). Treat it as a template:

- Rename the file to `CLAUDE.md`, change the heading, and replace each placeholder with your own block plugin name, theme name, and block namespace prefix.
- Drop sections that don't apply. The "no ACF" rule, the top border toggle, and the BEM shared components note are conventions from the source project. Keep them only if they fit yours.
- Keep the shape: stack, where things live, your project conventions, then a pointer to the figma-workflow skill.

The component and token maps under `project/figma-*.md` are also examples. Regenerate them by scanning your own block plugin and theme tokens, and keep them current as your design system grows. Stale maps lead Claude to invent components that already exist, which is the whole problem the maps are there to prevent.

## Watch your Figma MCP token usage

The Figma MCP runs as a long-lived server that stays connected for the whole session, not a one-off call. Once it's loaded, its tools and instructions sit in your context every turn, and the tools themselves return large payloads. A single `get_screenshot` can run into the thousands of tokens, and a `get_design_context` call on a nested frame can be larger again.

The figma-workflow skill already tries to keep this in check. The rules exist because the cost is real:

- Pull the component definition node, not the instance. Instances drag in their parent context, which inflates the response without adding useful information.
- Don't re-pull the same node to double-check something. You already have the result; reading it again is wasted tokens.
- Prefer `get_design_context` over `get_screenshot`. Screenshots are only worth fetching when the code output doesn't match the design and you need a visual reference to debug it.
- Skip `get_metadata` unless you actually need the layer hierarchy. Read the component map first and ask the user when you're not sure which block a Figma frame maps to.
- Resist the urge to call any Figma tool to "look around". The block name gate at the start of `/figma-workflow` exists so Claude doesn't burn tokens exploring before it knows what it's building.

If you find yourself in a long session that has drifted away from Figma work, starting a fresh session for the next task keeps the context clean.

## Example: comparing a Figma frame to an existing block

A useful first move with any new Figma frame is to ask Claude which existing block it maps to, before generating any code. This is the block name gate from `/figma-workflow` in plain prompt form. It stops Claude from inventing a parallel component when one already exists.

A prompt that works:

> Give me a comparison table of this Figma frame against the matching WP block, and tell me the block name. @https://www.figma.com/design/.../node-id=1234-5678

What Claude does with that:

1. Reads `project/figma-component-map.md` to look up candidate blocks.
2. Calls `mcp__figma__get_design_context` once on the node, with screenshots disabled to keep tokens down.
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

## Notes

The bundle assumes you're already running Claude Code with the official Figma plugin installed (it provides the `mcp__figma__*` tools the skills call). Install that first if it's missing.

Rules and skills with a `<!-- Last updated: ... -->` comment are tracked by a staleness check. If you see one older than about a month, review it before relying on it. AI best practices and tooling shift fast.
