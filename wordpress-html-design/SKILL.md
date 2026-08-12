---
name: wordpress-html-design
description: Design and build HTML/CSS pages (plugin admin screens, settings panels, dashboards, onboarding wizards, public-facing plugin pages) that comply with the WordPress design system, meet WCAG 2.1 AA accessibility, and are modern, cross-browser compatible, and responsive. Use this skill whenever the user is building a WordPress plugin or theme UI, an admin dashboard, a settings page, a wp-admin screen, or any HTML page meant to feel native inside WordPress — even if they just say "design a page," "build a dashboard," "make this look like WordPress," or "create the plugin settings UI." Also use it when reviewing or fixing an existing WordPress-adjacent HTML/CSS page for accessibility, responsiveness, or design-system compliance.
---

# WordPress HTML Page Design

Build HTML/CSS pages that a WordPress user would immediately recognize as belonging inside their dashboard — same color language, same spacing rhythm, same component behavior — while meeting a modern quality bar: accessible, responsive, cross-browser safe, and intuitive to use without instructions.

Treat this as designing for wp-admin (or a page styled to match it), not a generic marketing site. WordPress users have strong, specific expectations — a page that looks "modern" but ignores those conventions will feel foreign and reduce trust, which matters especially for security-sensitive plugin UIs.

## Before building

Establish these before writing code — infer from context where possible, ask only what's genuinely ambiguous:

1. **Context**: Is this a wp-admin screen (inside the WP chrome, sidebar/topbar already present), a full-width plugin page, a modal/panel, or a public-facing (non-admin) page? This changes which conventions apply.
2. **WordPress version baseline**: Assume current WordPress (6.x) unless told otherwise — this determines available admin color scheme values and component patterns.
3. **Content**: What does this page actually need to say and let the user do? Ground the layout in real content (see `frontend-design` skill if deeper visual-identity work is needed beyond WP conventions).
4. **Multisite context**: If relevant (see project context), note whether the page needs network-admin styling, which uses the same tokens but different navigation chrome.

## 1. WordPress Design System compliance

Read `references/wp-design-tokens.md` for the full token table (colors, spacing, typography, breakpoints, z-index, component classes) before writing CSS. Highlights:

- **Color**: WordPress admin's default scheme uses `#1d2327` (text), `#2271b1` (primary/links), `#f0f0f1` (background), `#dcdcde` (borders), `#d63638` (error/danger), `#00a32a` (success), `#dba617` (warning). Don't invent a new brand palette for admin screens — derive accents from these unless the user explicitly wants a distinct plugin identity, in which case keep structural chrome (backgrounds, borders, text) on WP tokens and reserve the custom palette for the plugin's own accent.
- **Typography**: WP admin's system font stack — `-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen-Sans, Ubuntu, Cantarell, "Helvetica Neue", sans-serif`. Base size 13–14px in admin contexts, not the 16px web default. Headings use existing admin scale, not arbitrary sizes.
- **Spacing**: WordPress admin is built on a loose 4px/8px rhythm (8, 12, 16, 20, 24, 32px are the common steps). Keep to this grid rather than arbitrary pixel values.
- **Components**: Reuse WP's own vocabulary and class conventions rather than reinventing them — buttons (`.button`, `.button-primary`, `.button-secondary`, `.button-link-delete`), notices (`.notice`, `.notice-success`, `.notice-error`, `.notice-warning`, dismissible pattern), form tables (`.form-table`), cards (`.postbox`), meta boxes, `.wrap` page container, `.nav-tab-wrapper` for tabbed settings pages. If the page will actually be enqueued inside wp-admin, use these real classes so it inherits core styles for free. If it's a standalone HTML mockup, replicate their visual behavior faithfully instead of inventing new patterns.
- **Icons**: Use Dashicons (WP's built-in icon font, class pattern `dashicons dashicons-<name>`) for anything in an admin context — don't mix in an unrelated icon set that will look foreign next to core UI.
- **Page chrome**: Respect the `.wrap` top padding, the placement of the page `<h1>` (once per page, immediately inside `.wrap`), and the admin notice area directly below it — this is where WP expects success/error messages to render.

## 2. Accessibility (WCAG 2.1 AA minimum)

Read `references/accessibility-checklist.md` for the full checklist. Non-negotiables to build in from the start, not retrofit:

- **Semantic HTML first**: `<button>` for actions, `<a>` only for navigation, real `<table>` markup for tabular data, `<fieldset>`/`<legend>` for grouped form controls, landmark elements (`<main>`, `<nav>`, `<header>`) even inside an admin screen fragment.
- **Labels**: Every form input has a real, associated `<label>` (or `aria-label`/`aria-labelledby` only when a visible label truly can't fit — e.g. an icon-only button). Never rely on placeholder text as a label.
- **Color contrast**: 4.5:1 minimum for body text, 3:1 for large text (18px+/14px+ bold) and meaningful UI components/icons. Check any custom accent color against its background — WP's own `#2271b1` on white passes; a custom brand color may not.
- **Keyboard**: Everything reachable and operable by keyboard alone, in a logical tab order. Visible focus indicators on every interactive element — never `outline: none` without an equally visible replacement.
- **Don't rely on color alone**: Status (error/success/warning) needs an icon or text label too, not just a colored border.
- **Motion**: Wrap non-essential animation/transitions in `@media (prefers-reduced-motion: reduce)`.
- **Alt text**: Meaningful images get descriptive `alt`; purely decorative images get `alt=""`.
- **Announce dynamic changes**: Content that updates without a page load (save confirmations, validation errors, live counters) needs `aria-live` regions so screen reader users aren't left behind.

## 3. Modern, maintainable CSS

- Use CSS custom properties for the token set (colors, spacing, radii) so the page can respond to WordPress's admin color scheme picker and to a plugin's own theme options without find-and-replace.
- Layout with Flexbox and Grid; avoid float-based layouts and fixed pixel-width containers.
- Use logical properties (`margin-inline`, `padding-block`, `inset-inline-start`) instead of physical ones (`margin-left`, `left`) so the page doesn't silently break for WordPress's RTL language users — this is a real, common WP audience, not an edge case.
- Fluid type/spacing with `clamp()` where a range makes sense, rather than a wall of breakpoint-specific overrides.
- Keep specificity flat and predictable — prefer single classes over deep descendant chains, and watch for a type selector (`.section`) and an element selector (`.cta`) canceling each other out on shared properties like padding/margin.
- Scope custom styles under a plugin-specific class/prefix so they can't leak into or be overridden by unrelated wp-admin chrome.

## 4. Cross-browser compatibility

- Target current stable Chrome, Firefox, Safari, and Edge; WordPress's own baseline is broadly evergreen browsers, but avoid bleeding-edge CSS (e.g. very new selectors) without a graceful fallback.
- Use `@supports` for progressive enhancement when using newer CSS features (`:has()`, container queries, subgrid) — provide a reasonable fallback layout for browsers that don't support them, don't let the page break.
- Test/mentally verify flexbox gap, `:focus-visible`, and form control styling specifically — these have historically had the most cross-browser inconsistency.
- Normalize form control appearance (`appearance: none` plus manual restyling) since native form elements render very differently per browser, which looks especially broken next to WP's own consistently-styled controls.

## 5. Responsive design

WordPress admin has established breakpoints — match them so a plugin page collapses/adapts in step with the rest of wp-admin, rather than at different points:

- **782px** — WP admin's primary responsive breakpoint; below this, the admin menu collapses to icons and admin pages typically stack to single-column.
- **600px** — secondary breakpoint for tighter mobile layouts.
- **480px** — smallest, for phone-width fine-tuning.

Design mobile-first: base styles for the smallest viewport, then layer up with `min-width` media queries. Ensure touch targets are at least 44×44px, tables either scroll horizontally or reflow to a card/definition-list pattern on narrow widths (don't let them overflow silently), and any multi-column dashboard layout has a defined single-column fallback.

## 6. Intuitive by default

- Follow WordPress's own interaction patterns so nothing needs explaining: primary action is the rightmost/most prominent button, destructive actions are visually distinct and usually require confirmation, settings pages use tabs or vertical nav the way core screens do, save state gives immediate feedback (WP's own notice pattern), unsaved changes are indicated before navigation away.
- Write interface copy in WordPress's own voice: plain, direct, active voice ("Save changes," not "Submit"), sentence case for labels and buttons, specific error messages that say what happened and how to fix it.
- One page, one clear primary job. If a settings page is trying to do five unrelated things, that's a sign it should be split into tabs or separate screens the way core does.

## Build process

1. Confirm context (admin vs. public, WP version baseline, real content) — assume sensibly rather than blocking on every detail.
2. Skim `references/wp-design-tokens.md` and pull the specific tokens/classes this page needs.
3. Draft semantic HTML structure first, no styling — confirm it makes sense read top-to-bottom with a screen reader before adding CSS.
4. Layer in CSS using the token system, mobile-first, with the WP breakpoints above.
5. Self-check against `references/accessibility-checklist.md` before calling it done.
6. Verify: does this look like it belongs in wp-admin? Does it work at 320px width and at desktop width? Does it work with keyboard only? Would the copy make sense to a non-technical WordPress site owner?

## Reference files

- `references/wp-design-tokens.md` — full WordPress admin color palette, typography scale, spacing system, breakpoints, and core CSS component classes to reuse.
- `references/accessibility-checklist.md` — WCAG 2.1 AA checklist scoped to WordPress admin-style pages, with the specific things that most often get missed.