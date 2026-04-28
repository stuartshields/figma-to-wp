# Figma → Code Component Map

Generated from a scan of `content/mu-plugins/wp-blocks/` and `content/themes/wp-theme/` on 2026-04-27.

This file is a lookup table. When Figma returns a design, grep for the closest match here **before** generating new markup. If something in Figma looks like an entry below, use the existing component — do not invent a parallel one.

Re-run the scan when blocks are added, renamed, or removed, or when new shared template-parts land in the theme.

---

## Gutenberg Blocks (wp-blocks plugin)

All blocks live under `content/mu-plugins/wp-blocks/blocks/<slug>/`. The block editor code is `edit.js` / `index.js` in that folder. The **frontend render** for most blocks is a template part in the theme at `content/themes/wp-theme/template-parts/blocks/<slug>.php` — the theme renders the block, not the plugin. This is important: when updating a block's markup, you usually edit the theme's render file, not the plugin.

### Hero blocks

**`wpb/primary-hero` — Primary Hero**
Full-height hero with background image, gradient overlays, heading, subheading, CTA buttons, and optional heart overlay. Used on the home page.
- Plugin: `wp-blocks/blocks/primary-hero/`
- Render: `wp-theme/template-parts/blocks/hero/primary-hero.php`
- Attributes: `align`, `backgroundImage`, `mobileImage`, `displayOption`, `heading`, `subheading`, `buttons`, `showHeartOverlay`
- Figma cue: Full-bleed tall hero, gradient over image, large heading + subhead + multiple CTAs, decorative heart element.

**`wpb/secondary-hero` — Secondary Hero**
Shorter hero for sub-pages with background image, gradient overlays, heading, and breadcrumbs.
- Plugin: `wp-blocks/blocks/secondary-hero/`
- Render: `wp-theme/template-parts/blocks/hero/secondary-hero.php`
- Attributes: `align`, `backgroundImage`, `mobileImage`, `displayOption`, `heading`, `subheading`
- Figma cue: Full-bleed shorter hero (~half height of primary), heading over image, breadcrumbs visible.

### Content blocks

**`wpb/callout-card` — Callout Card**
Long-form content block with profile image, heading, CTA button, and expandable body text.
- Plugin: `wp-blocks/blocks/callout-card/`
- Render: `wp-theme/template-parts/blocks/callout-card.php`
- Attributes: `align`, `heading`, `buttons`
- Figma cue: Wide-align card with profile/portrait image on one side, long body text that collapses with a "Read more" toggle.
- Uses: `template-parts/components/expandable-body.php` for the collapsible body.

**`wpb/callout-banner` — Callout Banner**
Full-width callout section with background image, eyebrow, heading, description, and CTA button on a navy background.
- Plugin: `wp-blocks/blocks/callout-banner/`
- Render: `wp-theme/template-parts/blocks/callout-banner.php`
- Attributes: `align`, `backgroundImage`, `eyebrow`, `heading`, `description`, `buttons`
- Figma cue: Full-width navy/dark banner with background image, eyebrow label + heading + CTA.
- Note: When this block is present on a page, the footer automatically switches to navy variant (see `footer.php`).

**`wpb/media-split` — Media Split**
Two-column image and text block with configurable image position and optional button.
- Plugin: `wp-blocks/blocks/media-split/`
- Render: `wp-theme/template-parts/blocks/media-split.php`
- Attributes: `align`, `image`, `eyebrow`, `heading`, `description`, `type`, `buttons`, `imagePosition` (left/right), `backgroundColor` (default `sea-salt`)
- Figma cue: Two-column layout, image on one side, text on the other, image position swappable.

**`wpb/media-split-vertical` — Media Split Vertical**
Stacked text-over-image content block with eyebrow, heading, and body text.
- Plugin: `wp-blocks/blocks/media-split-vertical/`
- Render: `wp-theme/template-parts/blocks/media-split-vertical.php`
- Attributes: `image`, `mobileImage`, `eyebrow`, `heading`, `description`, `displayOption`, `showBackground`
- Figma cue: Single column, text above or below image, stacked vertically (not a two-col layout).

**`wpb/info-grid` — Info Grid**
Five-panel grid highlighting key differentiators with icons, headings, and descriptions.
- Plugin: `wp-blocks/blocks/info-grid/`
- Render: `wp-theme/template-parts/blocks/info-grid.php`
- Attributes: `align`, `heading`, `buttons`, `panels` (array of 5 `{ heading, body }`)
- Figma cue: Five-panel grid (often 1 primary + 4 secondary), each panel with icon/graphic + heading + body.

**`wpb/nav-card-grid` — Nav Card Grid**
Navigational card grid with manually entered cards — title, image, link URL, optional description and link label.
- Plugin: `wp-blocks/blocks/nav-card-grid/`
- Render: `wp-theme/template-parts/blocks/nav-card-grid.php`
- Attributes: `cards`, `columns` (default 2), `heading`, `description`, `buttons`, `anchor`, `align`
- Figma cue: Grid of cards each with an image and title, each card is a link. Used for nav/category landing pages.

**`wpb/recipe-variant-selector` — Recipe Variant Selector**
Navy CTA card with editable heading/body and dynamic pill buttons linking to published scale variant child posts.
- Plugin: `wp-blocks/blocks/recipe-variant-selector/`
- Render: `wp-theme/template-parts/blocks/recipe-variant-selector.php`
- Attributes: `align`, `heading`, `body`
- Figma cue: Navy rounded card with heading on left, body text + numbered pill buttons on right. Called "CTA" in Figma.
- Buttons are fully dynamic — populated from published child/sibling posts with `wpb_recipe_scale` taxonomy terms.

**`wpb/anchor-links` — Anchor Links**
Hardcoded "Jump to:" navigation bar with smooth-scroll links to the WPRM recipe card and comments section. No editable attributes. Only used on child posts.
- Plugin: `wp-blocks/blocks/anchor-links/`
- Render: `wp-theme/template-parts/blocks/anchor-links.php`
- Attributes: none (anchor only)
- Figma cue: Sea-salt card with "Jump to:" label and pill buttons (Recipe, Comments & Questions) with down-arrow icons.

**`wpb/comments` — Comments**
Renders the post comments section. No editable attributes. Restricted to child posts (hidden from inserter on non-recipe post types and only renders when the current post has a `post_parent`).
- Plugin: `wp-blocks/blocks/comments/`
- Render: server-side via `comments_template()` inside the plugin's `render_callback` — **no theme template part**.
- Attributes: none
- Figma cue: Comments thread / "Comments & Questions" section on a recipe child post.

**`wpb/recipe-archive` — Recipe Archive**
Paginated grid of recipe cards with dietary tags, quantity selectors, and server-side pagination.
- Plugin: `wp-blocks/blocks/recipe-archive/`
- Render: `wp-theme/template-parts/blocks/recipe-archive.php`
- Attributes: `postsPerPage`, `anchor`, `align`
- Figma cue: 3-column grid of recipe cards with image, tags, "Make this recipe" button, "Read more" link, numbered pagination.
- Uses: `template-parts/components/recipe-card.php` for each card.

**`wpb/back-story` — Back Story**
Container-width content card with single image, heading, and rich body text.
- Plugin: `wp-blocks/blocks/back-story/`
- Render: `wp-theme/template-parts/blocks/back-story.php`
- Attributes: `image`, `heading`, `body`
- Figma cue: Sea-salt rounded card (820px max-width) with image on one side, heading + multi-paragraph body text on the other.

**`wpb/two-image` — Two Image**
Two side-by-side images with an optional shared caption.
- Plugin: `wp-blocks/blocks/two-image/`
- Render: `wp-theme/template-parts/blocks/two-image.php`
- Attributes: `caption`
- Figma cue: Two adjacent images, often with a single caption beneath them.

**`wpb/image` — Image**
Standalone responsive image block with separate desktop and mobile sources, optional caption, and configurable dimensions/aspect ratio.
- Plugin: `wp-blocks/blocks/image/`
- Render: `wp-theme/template-parts/blocks/image.php`
- Attributes: `align` (default `full`), `image`, `mobileImage`, `displayOption`, `caption`, `width`, `height`, `aspectRatio`, `sizeSlug`
- Figma cue: A single image (often full-bleed or wide) with potentially different desktop/mobile crops, optional caption beneath.
- Uses: `template-parts/components/art-direction-image.php`. Generates multiple `<source>` entries from the registered `image-block-*` image sizes.

**`wpb/image-carousel` — Image Carousel**
Full-width image carousel with heading, prev/next navigation, touch swipe, and keyboard support.
- Plugin: `wp-blocks/blocks/image-carousel/`
- Render: `wp-theme/template-parts/blocks/image-carousel.php`
- Attributes: `align`, `heading`, `images`
- Figma cue: Heading above a horizontally scrollable/carousel of images, with arrow controls.

**`wpb/service-carousel` — Service Carousel**
Horizontal carousel of service/recipe cards with image backgrounds, gradient overlay, prev/next arrows, and progress bar pagination.
- Plugin: `wp-blocks/blocks/service-carousel/`
- Render: `wp-theme/template-parts/blocks/service-carousel.php`
- Attributes: `align` (full), `postIds`, `eyebrow`, `heading`, `buttons`, `anchor`
- Figma cue: Eyebrow + heading above a row of tall image cards with text overlay at bottom, arrow controls and a progress bar below.

### FAQ blocks (nested)

**`wpb/faq` — FAQ (parent)**
Frequently asked questions with tabbed categories and accordion items.
- Plugin: `wp-blocks/blocks/faq/`
- Render: `wp-theme/template-parts/blocks/faq.php`
- Attributes: `align`, `heading`, `editorActiveTabIndex`
- Contains: one or more `wpb/faq-category` children.

**`wpb/faq-category` — FAQ Category (child of faq)**
A category tab containing FAQ items.
- Plugin: `wp-blocks/blocks/faq-category/`
- Parent: `wpb/faq` only
- Attributes: `title`, `uniqueId`
- Contains: one or more `wpb/faq-item` children.

**`wpb/faq-item` — FAQ Item (child of faq-category)**
A single question + answer accordion item.
- Plugin: `wp-blocks/blocks/faq-item/`
- Parent: `wpb/faq-category` only
- Attributes: `title`, `uniqueId`

### Profile / team blocks

**`wpb/profile-list` — Profile List**
Grid of team member profile cards (2 or 3 columns) with name, role, short description, and read more link. Opens modal on click.
- Plugin: `wp-blocks/blocks/profile-list/`
- Render: `wp-theme/template-parts/blocks/profile-list.php`
- Attributes: `align`, `profileIds`, `teamType` (default `executive`), `eyebrow`, `heading`, `description`, `buttons`, `anchor`
- Figma cue: Grid of team cards with portrait + name + role + truncated bio + "Read more".
- Uses: `template-parts/components/profile-modal.php` (include once per page).

**`wpb/profile-carousel` — Profile Carousel**
Carousel of team member profile cards with staggered fan layout on desktop and swipeable cards on mobile.
- Plugin: `wp-blocks/blocks/profile-carousel/`
- Render: `wp-theme/template-parts/blocks/profile-carousel.php`
- Attributes: `align` (full), `profileIds`, `heading`, `description`, `buttons`, `profileGradients`, `anchor`
- Figma cue: Full-bleed section with overlapping / fanned profile cards that the user can swipe through.
- Uses: `template-parts/components/profile-modal.php`.

**`wpb/profile-bio` — Profile Bio** (hidden, child-only)
Rich content area for a profile biography. Not in the inserter. Used inside profile post types.
- Plugin: `wp-blocks/blocks/profile-bio/`

**`wpb/profile-role` — Profile Role** (hidden, child-only)
Single-line role/title for a profile. Not in the inserter.
- Plugin: `wp-blocks/blocks/profile-role/`
- Attributes: `text`

**`wpb/profile-short-description` — Profile Short Description** (hidden, child-only)
Short description or tagline for a profile. Not in the inserter.
- Plugin: `wp-blocks/blocks/profile-short-description/`
- Attributes: `text`

### Marketing / metrics blocks

**`wpb/ticker-banner` — Ticker Banner**
Animated counter section with glass-morphism digit tiles on a background image.
- Plugin: `wp-blocks/blocks/ticker-banner/`
- Render: `wp-theme/template-parts/blocks/ticker-banner.php`
- Attributes: `align` (full), `backgroundImage`, `mobileImage`, `displayOption`, `eyebrow`, `heading`
- Figma cue: Full-bleed section with big digit tiles that animate/count up, glass/frosted effect, background image.

**`wpb/ticker-plain` — Ticker Plain**
Plain counter section with digit tiles on a light background.
- Plugin: `wp-blocks/blocks/ticker-plain/`
- Render: `wp-theme/template-parts/blocks/ticker-plain.php`
- Attributes: `align` (wide), `eyebrow`, `heading`
- Figma cue: Wide-align section with digit tiles, no background image, light surface. Simpler sibling of ticker-banner.

### Form blocks

**`wpb/contact-form` — Contact Form**
Two-column contact block with contact details on one side and an enquiry form on the other.
- Plugin: `wp-blocks/blocks/contact-form/`
- Render: `wp-theme/template-parts/blocks/contact-form.php`
- Attributes: `align`, `eyebrow`, `heading`, `description`, `addressLine1`, `addressLine2`, `email`, `instagramHandle`, `instagramUrl`, `privacyText`, `checkboxLabel`
- Figma cue: Two-column: contact info (address, email, Instagram) on one side, form fields + privacy checkbox on the other.

---

## Theme Template Parts

Template parts live in `content/themes/wp-theme/template-parts/`. The theme uses **`get_extended_template_part`** (parameterized template parts — variables are passed via an array and read as `$this->vars[...]` inside the template). This is different from vanilla `get_template_part` and is important to get right.

### Shared visual components (use these instead of writing new markup)

**`template-parts/components/animated-button.php` — Animated Button**
The canonical button component. Pill-shaped, with a sliding chevron on hover. **Use this any time Figma shows a button.** Do not hand-roll buttons.
- Include: `get_extended_template_part('components/animated-button', '', [ 'url' => '...', 'label' => '...', 'style' => 'solid', 'on_dark' => true, 'size' => 'default' ]);`
- Variants: `style` = `solid` (peach bg) | `transparent` (outline); when transparent, `on_dark` toggles white-on-dark vs plum-on-light border.
- Sizes: `default` (compact) | `responsive` (larger with responsive padding).
- Colours: solid = `bg-peach text-plum-700` hover `bg-peach-hover`. Outline on dark = `border-white/20`. Outline on light = `border-plum-700/20`.
- Figma cue: Any pill-shaped CTA with text + chevron icon.

**`template-parts/components/icon.php` — Icon**
Inline SVG icon renderer. Reads SVGs from `assets/images/icons/<name>.svg` and outputs them inline so they inherit `currentColor`. Cached per request.
- Include: `get_extended_template_part('components/icon', '', [ 'name' => 'chevron-right', 'class' => 'size-5' ]);`
- Figma cue: Any inline icon. Always use this — never write `<svg>` directly unless the icon is unique to one block.

**`template-parts/components/expandable-body.php` — Expandable Body**
Shared collapsible body component with overflow detection, divider, and "Read more / Read less" toggle. **Used by `callout-card`, `callout-banner`, and `media-split` blocks.**
- Include: `get_extended_template_part('components/expandable-body', '', [ 'content' => $html, 'collapsed_height_class' => 'max-h-...', 'unique_id' => $id, 'label_more' => 'Read more', 'label_less' => 'Read less', 'block_class' => 'wpb-expandable-body' ]);`
- Figma cue: Any body text that collapses to a fixed height with a "Read more" link below.

**`template-parts/components/art-direction-image.php` — Art Direction Image**
Renders a responsive `<picture>` element with desktop/mobile sources, or a plain `<img>` when only desktop exists. Handles art-directed responsive images.
- Include: `get_extended_template_part('components/art-direction-image', '', [ 'desktop_url' => '...', 'mobile_url' => '...', 'alt' => '...', 'breakpoint' => '1024px', 'class' => '', 'loading' => 'lazy', 'fetchpriority' => 'auto' ]);`
- Figma cue: Any image where Figma shows different desktop/mobile crops.

**`template-parts/components/profile-modal.php` — Profile Modal**
Modal shell for team member bios. **Include once per page** that has profile cards — the JS populates it when a card is clicked.
- Include: `get_template_part('template-parts/components/profile-modal');`
- Z-index: `z-60` (sits above the sticky header at `z-50`).
- Figma cue: Full-screen overlay modal with profile image + bio content, appears over profile grid/carousel.

**`template-parts/components/recipe-card.php` — Recipe Card**
Single recipe card with featured image, dietary tag pills, title, "Make this recipe" button (with quantity selector popover), and "Read more" link.
- Include: `get_extended_template_part('components/recipe-card', '', [ 'title' => '...', 'url' => '...', 'image_url' => '...', 'image_alt' => '...', 'tags' => [...], 'variants' => [...], 'card_id' => '...' ]);`
- Figma cue: Rounded card with food image top half, melon-light content bottom half, tag pills over image.

**`template-parts/components/breadcrumbs.php` — Breadcrumbs**
Breadcrumb nav with overflow dropdown. Only renders if your project's breadcrumb function (e.g. `WPBlocks\Core\get_breadcrumbs()`) returns items.
- Include: `get_template_part('template-parts/components/breadcrumbs');`
- Figma cue: Breadcrumb trail, usually inside the secondary hero.

**`template-parts/components/bg-pattern.php` — Background Pattern**
Decorative SVG swirl positioned absolutely behind block content, intentionally oversized so it bleeds into adjacent sections. Has dedicated mobile and desktop SVGs and one-off responsive positioning via a scoped `<style>` tag. Parent block must use `overflow-x-clip` to prevent horizontal scroll.
- Include: `get_extended_template_part('components/bg-pattern', '', [ 'class' => '...' ]);`
- Figma cue: Large decorative swirl/loop graphic behind a section's content, often offset to the right.

### Header template parts

**`template-parts/header/header.php` — Site Header**
The full site header: logo, desktop nav, mobile nav trigger. Handles transparent-over-hero vs solid-on-scroll via `data-nav-state`. Detects if the page starts with a hero block and adapts.
- Included by: root `header.php` via `get_template_part('template-parts/header/header')`.

**`template-parts/header/navigation-desktop-top-level.php`** — Top-level desktop nav items bar.
**`template-parts/header/navigation-desktop-panel.php`** — Desktop mega-menu dropdown panel.
**`template-parts/header/navigation-item-default.php`** — Default single nav item (desktop).
**`template-parts/header/navigation-item-rich-content.php`** — Rich-content nav item (desktop, with image/description).
**`template-parts/header/navigation-mobile.php`** — Mobile nav trigger/container.
**`template-parts/header/navigation-mobile-panel.php`** — Mobile slide-out panel.
**`template-parts/header/navigation-item-mobile-default.php`** — Default mobile nav item.
**`template-parts/header/navigation-item-mobile-rich-content.php`** — Rich-content mobile nav item.

All header partials are wired together via the main header template. Do not rebuild any of these from Figma — they are already the nav system.

### Footer template parts

**`template-parts/footer/navigation.php` — Footer Menu Column**
Renders a single menu column in the footer (About Us, How Can I Help?, etc.). Takes a menu slug and title via `get_extended_template_part`.
- Include: `get_extended_template_part('footer/navigation', '', [ 'menu' => 'footer-about-us', 'title' => 'About Us' ]);`
- Note: The footer itself (`footer.php` at theme root) is the container that includes multiple instances of this.

### Block render templates (not standalone — render files for blocks)

These files live in `template-parts/blocks/` and are the **frontend render output** for their matching `wpb/*` blocks. They are not reusable template parts — if you're editing one of these, you're editing a block's markup. See the Gutenberg Blocks section above for the pairing.

- `template-parts/blocks/callout-card.php` → `wpb/callout-card`
- `template-parts/blocks/callout-banner.php` → `wpb/callout-banner`
- `template-parts/blocks/contact-form.php` → `wpb/contact-form`
- `template-parts/blocks/faq.php` → `wpb/faq`
- `template-parts/blocks/image-carousel.php` → `wpb/image-carousel`
- `template-parts/blocks/info-grid.php` → `wpb/info-grid`
- `template-parts/blocks/media-split.php` → `wpb/media-split`
- `template-parts/blocks/media-split-vertical.php` → `wpb/media-split-vertical`
- `template-parts/blocks/nav-card-grid.php` → `wpb/nav-card-grid`
- `template-parts/blocks/profile-carousel.php` → `wpb/profile-carousel`
- `template-parts/blocks/profile-list.php` → `wpb/profile-list`
- `template-parts/blocks/service-carousel.php` → `wpb/service-carousel`
- `template-parts/blocks/ticker-banner.php` → `wpb/ticker-banner`
- `template-parts/blocks/ticker-plain.php` → `wpb/ticker-plain`
- `template-parts/blocks/back-story.php` → `wpb/back-story`
- `template-parts/blocks/two-image.php` → `wpb/two-image`
- `template-parts/blocks/image.php` → `wpb/image`
- `template-parts/blocks/hero/primary-hero.php` → `wpb/primary-hero`
- `template-parts/blocks/hero/secondary-hero.php` → `wpb/secondary-hero`
- `template-parts/blocks/recipe-archive.php` → `wpb/recipe-archive`
- `template-parts/blocks/hero/partials/hero-gradients.php` → shared gradient overlay partial used by both hero render files
- `template-parts/blocks/dummy-dynamic.php` → dev/placeholder render for dynamic block testing

---

## Theme Root Files

**`header.php`** — Minimal root header. Opens HTML, runs `wp_head()`, then includes `template-parts/header/header.php`.
**`footer.php`** — Full footer markup (sea-salt bordered card with logo, menu columns, Instagram link, copyright, ornament). Auto-switches to navy background when `wpb/callout-banner` is on the page.
**`index.php`** — Main fallback template. Standard WP loop wrapped in `<main class="flex-1">` with `get_header()` / `get_footer()`.
**`functions.php`** — Bootstrap (loads `inc/` namespaces, theme setup). Not a visual template.

No `single.php`, `archive-*.php`, or `page-*.php` at the theme root — this is a block-based theme, page rendering happens via blocks in the editor.

---

## Plugin Editor Components (wp-blocks/components/)

These are **editor-side JSX helpers** used only inside the block editor UI (sidebar controls, appenders, custom toolbars). They do **not** render on the frontend. Listed for completeness, but almost never relevant when translating Figma designs — Figma shows the frontend output, these are editor-only utilities.

- `button-group-editor/` — Button group editor (manages `buttons` array attribute across blocks that use CTAs)
- `button-link/` — Button link chooser (editor inspector control)
- `button-style-toggle/` — Toggle between solid/transparent button styles
- `image-controls/` — Media library image picker + controls
- `art-direction-controls/` — Desktop/mobile image controls for art-directed responsive images
- `post-item/` — Editor preview of a post item (used in carousels/lists)
- `post-item/gradients.js` — Gradient overlay helpers for post-item
- `post-selector/` — Post/profile picker UI
- `post-grid/` — Generic post/profile grid editor preview (used by carousels and lists)
- `profile-modal/` — Profile modal editor preview
- `accordion-appender/` — Custom appender for FAQ accordion blocks
- `sortable/` — Drag-to-reorder wrapper (used in FAQ, panels, etc.)
- `tabs/` — Editor tabs primitive (context, tablist, tab, tabpanel) used in FAQ block
- `ticker-animation/` — Ticker number animation helper

These are maintenance targets when adding new blocks or editor controls, not when implementing designs.

---

## Guidance: Mapping Figma → This Codebase

When Figma returns a design:

1. **Is it a full section?** → Probably a block. Grep `wp-blocks/blocks/*/block.json` for a title/description match. Render edits happen in `wp-theme/template-parts/blocks/`, editor UI edits happen in `wp-blocks/blocks/<slug>/`.
2. **Is it a button?** → Always `animated-button.php`. Never write a new button.
3. **Is it an icon?** → Always `components/icon.php`. Drop the SVG into `assets/images/icons/` if it doesn't exist there yet.
4. **Is it an image with art direction (different desktop/mobile crops)?** → `components/art-direction-image.php`.
5. **Is it a body text block that collapses with "Read more"?** → `components/expandable-body.php`.
6. **Is it a modal showing a team member bio?** → `components/profile-modal.php` — already exists, don't rebuild it.
7. **Is it nav (header or footer)?** → Existing partials in `template-parts/header/` and `template-parts/footer/` — modify, don't recreate.
8. **None of the above?** → It's probably net-new. Confirm with the user before scaffolding a block or standalone partial.
