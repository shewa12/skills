# WordPress Admin Design Tokens

Reference values for building HTML/CSS that matches WordPress's native admin design system (wp-admin). Values reflect the default admin color scheme in current WordPress (6.x). If the target site uses a different admin color scheme (Light, Blue, Coffee, Ectoplasm, Midnight, Ocean, Sunrise), the accent colors change but the structural tokens (spacing, typography, breakpoints) stay the same.

## Color

### Default scheme
| Token | Hex | Use |
|---|---|---|
| `--wp-text` | `#1d2327` | Primary body/heading text |
| `--wp-text-secondary` | `#50575e` | Secondary/muted text |
| `--wp-link` | `#2271b1` | Links, primary accent |
| `--wp-link-hover` | `#135e96` | Link/accent hover state |
| `--wp-bg` | `#f0f0f1` | Page background |
| `--wp-surface` | `#ffffff` | Card/panel background |
| `--wp-border` | `#dcdcde` | Default borders/dividers |
| `--wp-border-focus` | `#2271b1` | Focus ring / active border |
| `--wp-success` | `#00a32a` | Success states |
| `--wp-warning` | `#dba617` | Warning states |
| `--wp-error` | `#d63638` | Error/destructive states |
| `--wp-admin-bar-bg` | `#1d2327` | Top admin bar background |
| `--wp-menu-bg` | `#1d2327` | Sidebar menu background |
| `--wp-menu-highlight` | `#2271b1` | Active sidebar item |

### Contrast notes
- `#2271b1` on `#ffffff` ≈ 4.6:1 — passes AA for normal text.
- `#50575e` on `#ffffff` ≈ 6.4:1 — safe for secondary text.
- Never place white text on `--wp-warning` (`#dba617`) — insufficient contrast; use `--wp-text` on warning backgrounds instead.

## Typography

- Font stack: `-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen-Sans, Ubuntu, Cantarell, "Helvetica Neue", sans-serif`
- Base body size in admin: `13px`–`14px` (not the 16px web default — admin UI is denser)
- Page title (`<h1>` inside `.wrap`): `23px`, normal weight
- Section headings (`<h2>`): `18px`–`20px`
- Line height: `1.4`–`1.5` for body text
- Font weight: `400` body, `600` for emphasis/labels — avoid `700`+ except sparingly

## Spacing

Loose 4/8px rhythm. Common steps: `4, 8, 12, 16, 20, 24, 32, 40px`. Keep margins/padding on this grid rather than arbitrary values.

- `.wrap` top padding: `20px` (below the admin bar/header)
- Standard card/postbox padding: `12px 16px` (top) with body padding `12px`
- Form row spacing (`.form-table`): `15px` vertical padding per row

## Breakpoints

| Width | Behavior |
|---|---|
| `782px` | Primary breakpoint — admin menu collapses to icon-only, admin pages typically drop to single column |
| `600px` | Secondary breakpoint — tighter mobile adjustments |
| `480px` | Smallest — phone-width fine tuning |

Build mobile-first (`min-width` queries layering up from a single-column base).

## Core CSS classes to reuse (when the page will run inside wp-admin)

| Class | Purpose |
|---|---|
| `.wrap` | Standard page container — always wrap admin page content in this |
| `.button`, `.button-primary`, `.button-secondary` | Buttons — primary is the filled blue action, secondary is outlined |
| `.button-link-delete` | Destructive text-link-styled action |
| `.notice`, `.notice-success`, `.notice-error`, `.notice-warning`, `.notice-info`, `.is-dismissible` | Admin notice banners — render directly below the `<h1>` |
| `.form-table` | Standard two-column label/field settings table |
| `.postbox` | Card/meta-box container (used on dashboard and edit screens) |
| `.nav-tab-wrapper`, `.nav-tab`, `.nav-tab-active` | Tabbed navigation for settings pages |
| `.widefat` | Full-width, striped admin table (e.g. `.widefat.striped`) |
| `dashicons dashicons-<name>` | Icon font — use for any admin-context icon |
| `.screen-reader-text` | Visually-hidden but screen-reader-accessible text (WP's own sr-only utility) |

## Z-index landmarks (avoid colliding with core chrome)

- Admin bar: `z-index: 99999`
- Admin menu (sidebar): `z-index: 9990`
- Modals/dialogs you introduce should generally exceed these (e.g. `100000+`) only if they're meant to sit above the admin bar; otherwise stay below it.