# Accessibility Checklist — WCAG 2.1 AA for WordPress-style pages

Work through this before considering a page done. Grouped by WCAG principle. Items marked ★ are the ones most often missed on WordPress admin/plugin pages specifically.

## Perceivable

- [ ] All meaningful images have descriptive `alt` text; purely decorative images/icons use `alt=""` or `aria-hidden="true"`.
- [ ] Text contrast is at least 4.5:1 (normal text) or 3:1 (large text: 18px+/14px+ bold).
- [ ] ★ UI component and icon contrast is at least 3:1 against adjacent colors (form field borders, icon-only buttons, status dots) — this is checked far less often than text contrast.
- [ ] Color is never the only way status/meaning is conveyed — pair with icon, text, or pattern (e.g. an error isn't only a red border; it also has an error icon/message).
- [ ] Content reflows without horizontal scrolling or loss of function down to 320px width, and up to 400% zoom.
- [ ] Text can be resized to 200% without breaking layout or clipping content.

## Operable

- [ ] Every interactive element is reachable via keyboard alone (Tab/Shift+Tab), in a logical order matching the visual layout.
- [ ] ★ Every focusable element has a visible focus indicator — never `outline: none` without a replacement of equal or greater visibility. Check this specifically on custom-styled buttons, tabs, and dropdown triggers, which are the most common place it gets silently stripped.
- [ ] No keyboard trap — a user can always Tab out of any widget (modal, dropdown, date picker).
- [ ] Skip link to main content is present on full pages (not required inside a wp-admin screen fragment, since core already provides one).
- [ ] Touch targets are at least 44×44px (or have adequate spacing if visually smaller).
- [ ] Time limits (session timeouts, auto-refreshing content) can be paused, extended, or disabled — relevant for security plugins with scan timers/countdowns.
- [ ] ★ Animations/auto-playing content respect `@media (prefers-reduced-motion: reduce)`.

## Understandable

- [ ] Every form input has a real associated `<label>` — `for`/`id` pair, or the label wraps the input. Placeholder text is never the only label.
- [ ] Required fields are marked in a way conveyed to assistive tech (`required` attribute or `aria-required="true"`, not just a visual asterisk).
- [ ] Error messages are specific and actionable ("Enter a valid email address," not "Invalid input"), and are programmatically associated with their field (`aria-describedby`).
- [ ] Labels, icons, and interaction patterns are used consistently across the page (a save icon always means save, not sometimes save and sometimes refresh).
- [ ] Navigation and layout structure stays consistent across the plugin's own screens.

## Robust

- [ ] Semantic HTML is used before ARIA — `<button>` not `<div onclick>`, real `<table>` for tabular data, `<nav>`/`<main>`/`<header>` landmarks.
- [ ] ARIA is only added where semantic HTML can't express the pattern (e.g. `aria-expanded` on a custom disclosure toggle), and ARIA attributes are kept in sync with actual state.
- [ ] ★ Dynamic content updates (save confirmations, live validation, async-loaded results, progress/scan status) are announced via `aria-live="polite"` (or `assertive` for urgent errors) — easy to miss because it works fine visually and only breaks for screen reader users.
- [ ] Heading levels are sequential and describe actual document structure (one `<h1>` per page, no skipped levels chosen for visual size instead of structure).
- [ ] Custom widgets (tabs, accordions, modals, tooltips) follow the applicable WAI-ARIA Authoring Practices pattern for expected keyboard behavior, not an invented interaction.

## Quick manual test pass

1. Unplug the mouse — complete the page's primary task with keyboard only.
2. Turn on a screen reader (VoiceOver/NVDA) and listen through the page top to bottom — does it make sense without seeing it?
3. Zoom browser to 200% — does anything clip, overlap, or become unusable?
4. Toggle OS "reduce motion" — does anything still animate that shouldn't?
5. Check every status color (success/warning/error) against its background with a contrast checker.

## Useful references

- WCAG 2.1 quick reference: https://www.w3.org/WAI/WCAG21/quickref/
- WAI-ARIA Authoring Practices (component patterns): https://www.w3.org/WAI/ARIA/apg/patterns/
- WordPress accessibility coding standards: https://make.wordpress.org/core/handbook/best-practices/coding-standards/accessibility-coding-standards/