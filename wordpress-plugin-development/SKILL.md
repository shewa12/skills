---
name: wp-plugin-architecture
description: Architect and write WordPress plugins using a feature-based structure (core / inc / assets / src / templates / views), PSR-4 Composer autoloading, WPCS-compliant code, PSR-12 class naming, and WordPress plugin security standards. Use this skill whenever the user wants to create, scaffold, extend, refactor, review, or audit a WordPress plugin, add a feature/module/admin page/REST endpoint/shortcode/Gutenberg block/custom post type/settings page to a plugin, or asks about WordPress hooks, nonces, sanitization, escaping, capabilities, or plugin structure - even if they do not say "architecture" or "best practices".
---

# WordPress Plugin Architecture

Build plugins that are modular, secure, and predictable. The structure below exists so that any feature can be found, tested, and removed without touching unrelated code, and so that a reviewer (or the wordpress.org plugin team) can verify security by reading each layer separately.

Before writing code, confirm with the user (or infer from context): plugin name, slug, text domain, vendor namespace, minimum WP and PHP versions, and the list of features. Then scaffold, then implement features one at a time.

---

## 1. Directory structure

```
plugin-slug/
├── plugin-slug.php          # Bootstrap only: header, constants, autoload, boot
├── uninstall.php            # Cleanup on delete
├── composer.json
├── phpcs.xml.dist
├── readme.txt
├── languages/               # .pot / .po / .mo
├── core/                    # Framework-level plumbing (no business logic)
│   ├── Plugin.php           # Singleton main class
│   ├── Container.php        # Dependency container / service resolver
│   ├── Activator.php
│   ├── Deactivator.php
│   ├── Loader.php           # Registers hooks of every feature
│   └── Contracts/           # Interfaces (e.g. Hookable.php, Feature.php)
├── inc/                     # Every feature lives in its own folder
│   ├── Feature1/
│   │   ├── Controller.php   # Registers hooks, handles request, calls service
│   │   ├── Service.php      # Business logic
│   │   └── Model.php        # Data access (CPT, meta, options, custom table)
│   └── Feature2/
│       ├── Controller.php
│       ├── Service.php
│       └── Model.php
├── assets/
│   ├── css/
│   └── js/
├── src/                     # Only if React / Gutenberg source exists
│   └── (built output goes to build/ via @wordpress/scripts)
├── templates/               # Frontend output templates (overridable)
└── views/                   # Admin-side UI files
```

Rules for the structure:

- **Bootstrap file contains no logic.** It defines constants, requires `vendor/autoload.php`, and calls `Plugin::instance()->boot()`. Keeping it thin means the plugin can be loaded in tests and CLI without side effects.
- **`core/` knows nothing about features.** If a class mentions a specific feature, it belongs in `inc/`.
- **`inc/<Feature>/` is self-contained.** A feature may depend on `core/` and on other features' *services* through the container, never on another feature's controller or model directly. Deleting a feature folder plus its registration line should leave the plugin working.
- **Add only the layers a feature needs.** A feature with no persistence has no Model. Never create empty classes to satisfy the pattern.
- **Layer responsibilities** (this is what keeps the code reviewable):
  - *Controller*: hook registration, reading and validating request input, nonce and capability checks, choosing a response (render a view/template, JSON, redirect). No business logic, no SQL.
  - *Service*: business rules. Receives already-sanitized, typed values. No `$_POST`/`$_GET` access, no output.
  - *Model*: reads and writes data (`WP_Query`, `$wpdb->prepare`, options, meta). No hooks, no output.
- **Templates vs views**: `templates/` is public-facing output and should be overridable from the theme (`yourtheme/plugin-slug/template-name.php`) through a loader helper. `views/` is admin UI and is not overridable.
- **Folder casing**: top-level folders are lowercase (`core`, `inc`, `assets`, `src`, `templates`, `views`). Feature folders under `inc/` use StudlyCaps so they map 1:1 to PSR-4 namespaces.

---

## 2. Composer and PSR-4 autoloading

All classes are loaded by Composer. Never use `require`/`include` to load class files, and never write a custom `spl_autoload_register`.

```json
{
  "name": "vendor/plugin-slug",
  "type": "wordpress-plugin",
  "require": { "php": ">=7.4" },
  "require-dev": {
    "wp-coding-standards/wpcs": "^3.0",
    "dealerdirect/phpcodesniffer-composer-installer": "^1.0",
    "phpcompatibility/phpcompatibility-wp": "^2.1",
    "phpstan/phpstan": "^1.10",
    "szepeviktor/phpstan-wordpress": "^1.3",
    "phpunit/phpunit": "^9.6",
    "yoast/phpunit-polyfills": "^2.0"
  },
  "autoload": {
    "psr-4": {
      "Vendor\\PluginSlug\\Core\\": "core/",
      "Vendor\\PluginSlug\\Inc\\": "inc/"
    }
  },
  "config": {
    "optimize-autoloader": true,
    "allowed-plugins": { "dealerdirect/phpcodesniffer-composer-installer": true }
  },
  "scripts": {
    "lint": "phpcs",
    "fix": "phpcbf",
    "analyse": "phpstan analyse"
  }
}
```

- Namespace root is `Vendor\PluginSlug` (StudlyCaps). Sub-namespaces mirror the folder path exactly: `inc/Orders/Service.php` is `Vendor\PluginSlug\Inc\Orders\Service`.
- Ship `vendor/` with the release build (run `composer install --no-dev -o`). Add a friendly admin notice if `vendor/autoload.php` is missing instead of fataling.
- Prefix or scope third-party runtime dependencies (Strauss or PHP-Scoper) to avoid version collisions with other plugins.

---

## 3. Coding standards: WPCS plus PSR-12 naming

Two standards apply, with a clear split so they never conflict:

| Concern | Standard | Example |
|---|---|---|
| Class, interface, trait, enum **names** | PSR-12 (StudlyCaps) | `OrderService` |
| Class **file names** | PSR-12 / PSR-4 (match class name) | `OrderService.php` |
| Namespaces | PSR-4 / StudlyCaps | `Vendor\PluginSlug\Inc\Orders` |
| Everything else: indentation (tabs), spacing in parentheses, Yoda conditions, array syntax, braces, docblocks | **WPCS** | `if ( 'draft' === $status ) {` |
| Function, method, variable, hook names | **WPCS** (snake_case) | `get_order_total()`, `$order_id` |
| Constants | WPCS (UPPER_SNAKE) | `PLUGIN_SLUG_VERSION` |

Why methods are snake_case: the user-specified rule limits PSR-12 to class names and file names, and WPCS governs the rest, which keeps method names consistent with the WordPress API they sit beside.

Required practices:

- Every file starts with `<?php`, then `namespace`, `use` statements, and an `ABSPATH` guard when the file is not a pure class definition: `defined( 'ABSPATH' ) || exit;`. Class-only files that are autoloaded may omit the guard because they produce no output, but templates and views must include it.
- One class per file. File name equals class name exactly (`class-foo.php` style names are **not** used here).
- Declare `declare( strict_types=1 );` only after confirming every caller passes correctly typed values; WordPress core passes strings from the database freely, so be careful at hook boundaries.
- Use typed properties, parameter types, and return types (PHP 7.4+). Use `?Type` for nullable values.
- Write full docblocks for classes, methods, properties, hooks (`@since`, `@param`, `@return`). Document every custom action and filter with a docblock directly above `do_action`/`apply_filters`.
- Prefix every global: constants, option names, meta keys, transients, hook names, script handles, CPT/taxonomy slugs, REST namespaces, table names. Use `plugin_slug_` for snake_case and `plugin-slug` for handles/slugs.
- Keep methods short (aim for under ~30 lines) and classes focused. Prefer composition over inheritance.
- Use `phpcs.xml.dist` with the `WordPress` ruleset, set `text_domain`, `prefixes`, and `minimum_supported_wp_version`, and exclude the file-name sniffs that would demand `class-*.php` names:

```xml
<?xml version="1.0"?>
<ruleset name="Plugin Slug">
  <file>.</file>
  <exclude-pattern>*/vendor/*</exclude-pattern>
  <exclude-pattern>*/node_modules/*</exclude-pattern>
  <exclude-pattern>*/build/*</exclude-pattern>
  <arg name="extensions" value="php"/>
  <arg value="ps"/>
  <rule ref="WordPress">
    <exclude name="WordPress.Files.FileName.InvalidClassFileName"/>
    <exclude name="WordPress.Files.FileName.NotHyphenatedLowercase"/>
  </rule>
  <rule ref="WordPress.WP.I18n">
    <properties><property name="text_domain" type="array"><element value="plugin-slug"/></property></properties>
  </rule>
  <rule ref="WordPress.NamingConventions.PrefixAllGlobals">
    <properties><property name="prefixes" type="array"><element value="plugin_slug"/><element value="Vendor\PluginSlug"/></property></properties>
  </rule>
  <config name="minimum_supported_wp_version" value="6.2"/>
  <rule ref="PHPCompatibilityWP"/>
  <config name="testVersion" value="7.4-"/>
</ruleset>
```

---

## 4. Core layer

### Singleton main class

```php
<?php
namespace Vendor\PluginSlug\Core;

defined( 'ABSPATH' ) || exit;

final class Plugin {

	private static ?Plugin $instance = null;

	private Container $container;

	public static function instance(): Plugin {
		if ( null === self::$instance ) {
			self::$instance = new self();
		}
		return self::$instance;
	}

	private function __construct() {
		$this->container = new Container();
	}

	private function __clone() {}

	public function __wakeup() {
		throw new \Exception( 'Cannot unserialize singleton.' );
	}

	public function boot(): void {
		$this->container->register_defaults();
		add_action( 'plugins_loaded', array( $this, 'init' ) );
	}

	public function init(): void {
		( new Loader( $this->container ) )->register_features();
	}

	public function container(): Container {
		return $this->container;
	}
}
```

Use the singleton only for the main class (and the container). Feature classes are plain objects resolved from the container; singletons everywhere make testing painful.

### Container

- Maps an identifier (the class name) to a factory closure, resolves lazily, and caches shared instances.
- Constructor injection only. Do not call `Plugin::instance()->container()->get()` from inside services (service-locator anti-pattern); inject what is needed.
- Keep it small: `set()`, `get()`, `has()`, `share()`. Reach for PHP-DI only if the plugin really needs autowiring.

### Loader and feature registration

- A feature controller implements a `Hookable` contract with a single `register(): void` method that adds its actions and filters.
- `Loader` holds the list of controller class names, resolves each from the container, and calls `register()`. Adding a feature is one line in that list.
- Load admin-only controllers only when `is_admin()` and frontend-only controllers only when not, so no unused code runs on a request.

### Lifecycle classes

- `Activator::activate()`: check PHP/WP minimums (bail with `wp_die()` and a clear message), create tables with `dbDelta()`, set default options with `add_option()`, store the version, schedule cron events, and flush rewrite rules **after** registering CPTs.
- `Deactivator::deactivate()`: clear scheduled events and flush rewrite rules. Never delete user data here.
- `uninstall.php`: verify `defined( 'WP_UNINSTALL_PLUGIN' )`, then remove options, transients, custom tables, user meta, and CPT data, honoring a user setting like "Delete data on uninstall" when data loss is significant. Handle multisite by looping through sites.

---

## 5. Feature pattern (`inc/<Feature>/`)

```php
<?php
namespace Vendor\PluginSlug\Inc\Orders;

use Vendor\PluginSlug\Core\Contracts\Hookable;

defined( 'ABSPATH' ) || exit;

final class Controller implements Hookable {

	public function __construct( private Service $service ) {} // PHP 8; use property assignment on 7.4

	public function register(): void {
		add_action( 'admin_post_plugin_slug_save_order', array( $this, 'handle_save' ) );
	}

	public function handle_save(): void {
		check_admin_referer( 'plugin_slug_save_order', 'plugin_slug_nonce' );

		if ( ! current_user_can( 'manage_options' ) ) {
			wp_die( esc_html__( 'You are not allowed to do this.', 'plugin-slug' ), 403 );
		}

		$order_id = isset( $_POST['order_id'] ) ? absint( wp_unslash( $_POST['order_id'] ) ) : 0;
		$note     = isset( $_POST['note'] ) ? sanitize_textarea_field( wp_unslash( $_POST['note'] ) ) : '';

		$result = $this->service->save( $order_id, $note );

		wp_safe_redirect( add_query_arg( 'saved', (int) $result, wp_get_referer() ) );
		exit;
	}
}
```

Service and Model receive typed, sanitized values and return typed results (value objects, arrays, or `\WP_Error`). Models are the only place that touches `$wpdb` and `update_post_meta()`.

---

## 6. Security (WordPress plugin security standard)

Treat all input as hostile and all output as unsafe. Apply this checklist to every controller, template, and view. The mantra is: **validate/sanitize on input, escape on output, verify intent (nonce) and permission (capability) before acting.**

**Direct access**
- Add `defined( 'ABSPATH' ) || exit;` to every file that can execute or output on its own (bootstrap, templates, views, includes without classes).
- Add an `index.php` (`<?php // Silence is golden.`) to directories only if the server may list them.

**Authentication and authorization**
- Nonce checks prove intent; they do **not** prove permission. Always do both.
- Use `check_admin_referer()` / `wp_verify_nonce()` for forms, `check_ajax_referer()` for AJAX, and a REST `permission_callback` for REST (the `X-WP-Nonce` header is validated by core for cookie auth).
- Use the narrowest capability that fits (`edit_post` with a post ID rather than `manage_options` when appropriate). Never use `is_admin()` as a permission check; it only tells you the request is for an admin screen.
- Register privileged AJAX only on `wp_ajax_{action}`. Add `wp_ajax_nopriv_{action}` only when the action is genuinely public, and then treat it as an untrusted endpoint.
- REST routes must always set `permission_callback`. Use `'__return_true'` only for intentionally public data, and say so in a comment.

**Input: validate, then sanitize, and always `wp_unslash()` first**
- Check `isset()`/type before use; reject values outside an allowlist (enums, `in_array( $v, $allowed, true )`).
- Sanitizers: `sanitize_text_field`, `sanitize_textarea_field`, `sanitize_email`, `sanitize_key`, `sanitize_title`, `sanitize_file_name`, `esc_url_raw`, `absint`, `intval`, `floatval`, `wp_kses_post`/`wp_kses` for limited HTML, `rest_sanitize_boolean`.
- Never read raw `$_REQUEST`; read the specific superglobal and key you need.
- For REST, declare `args` with `type`, `required`, `validate_callback`, and `sanitize_callback` for every parameter. For settings, use `register_setting()` with `sanitize_callback` and `show_in_rest` schema.
- Validate file uploads with `wp_check_filetype_and_ext()`, size limits, and `wp_handle_upload()`. Never trust `$_FILES['type']`.

**Output: escape late, as close to the echo as possible**
- `esc_html()`, `esc_attr()`, `esc_url()`, `esc_js()`, `esc_textarea()`, `wp_kses_post()`, `wp_json_encode()`; `esc_html__()`/`esc_html_e()`/`esc_attr__()` for translated strings.
- Escape on output even when the value was sanitized on input; the data may have come from the database, another plugin, or an import.
- Pass data to JavaScript with `wp_localize_script()` / `wp_add_inline_script()` plus `wp_json_encode()`, never by concatenating into a script string.
- Late-escaped output must be visible to PHPCS. If a helper returns pre-escaped HTML, use `wp_kses()` with an explicit allowlist, or document with `// phpcs:ignore WordPress.Security.EscapeOutput -- reason`. Every ignore comment needs a reason.

**Database**
- Always use `$wpdb->prepare()` for any query with variables. Use `%i` for identifiers on WP 6.2+, or validate table/column names against an allowlist.
- Use `$wpdb->insert/update/delete` with format arrays where possible. Use `esc_like()` with `LIKE`. Build `IN ()` lists with generated placeholders.
- Prefer `WP_Query`, `get_posts`, and meta/term APIs over raw SQL.

**Other**
- Redirect with `wp_safe_redirect()` and `exit` afterwards. Validate user-supplied URLs with `wp_http_validate_url()` before remote requests; use `wp_safe_remote_get/post` for user-influenced URLs (SSRF protection).
- Never use `eval()`, `unserialize()` on user input (use JSON), `extract()`, `create_function`, or shell execution functions.
- Do not store secrets in code or in publicly readable options. Mask them in the UI, and never log them.
- Keep user-uploaded or generated files in `wp_upload_dir()` subfolders with an `.htaccess`/`index.php` deny when files must not be served directly.
- Do not echo raw exception messages or SQL errors to users; log with `error_log()` only when `WP_DEBUG_LOG` is enabled.
- Honor privacy: register data with the Privacy API (`wp_add_privacy_policy_content`, exporters/erasers) when personal data is stored.
- Load third-party scripts and fonts locally; do not call external services without disclosure in `readme.txt` and user consent.

---

## 7. Assets

- Register in one place per feature, enqueue only where needed (`admin_enqueue_scripts` with a `$hook_suffix` check; frontend only when the shortcode/block/template is present).
- Always pass a version (`PLUGIN_SLUG_VERSION` or the build asset file's version) and declare dependencies. Use `wp_enqueue_script()` with `array( 'strategy' => 'defer', 'in_footer' => true )` on WP 6.3+.
- Ship minified files for production and keep sources unminified in the repo (`assets/css/admin.css` and `admin.min.css`), loading `.min` unless `SCRIPT_DEBUG` is on.
- Use handles prefixed with the plugin slug. Namespace CSS classes (`.plugin-slug-card`) and scope admin styles to your own screens so you never restyle other plugins.
- No inline `<script>`/`<style>` in PHP files; use `wp_add_inline_*` when a small inline snippet is unavoidable.
- Use jQuery only when needed; prefer vanilla JS or `@wordpress/*` packages. Do all JS network calls through `wp.apiFetch` with REST nonces.

---

## 8. React and Gutenberg (`src/`)

- Use `@wordpress/scripts` (`wp-scripts build/start`) and output to `build/`. Load the generated `*.asset.php` for dependencies and version.
- Each block has a `block.json` (apiVersion 3, name `vendor/block-name`, textdomain, `render` for dynamic blocks). Register with `register_block_type( __DIR__ . '/build/blocks/name' )`.
- Dynamic blocks render through a PHP render callback that loads a file from `templates/`; escape all output in it.
- Do not put secrets or privileged logic in JS. Server-side code re-validates every REST request.
- Admin React apps mount on a single container element rendered by a file in `views/`, with data passed via `wp_add_inline_script`/REST, not scattered globals.

---

## 9. Templates and views

- A template or view contains markup and light control flow (`if`, `foreach`) only. All data arrives as already-prepared variables from the controller; no queries, no `$_GET`, no business rules.
- Use the alternative syntax (`if (): ... endif;`) in templates for readability.
- Render through a helper that extracts nothing implicitly: `Template::render( 'single/card', array( 'title' => $title ) )`, which resolves the theme override path first, then falls back to `templates/`. Validate the template name against `[a-z0-9/_-]` to prevent path traversal.
- Admin views start with `defined( 'ABSPATH' ) || exit;`, then a capability check in the controller that renders them, `settings_fields()`/`wp_nonce_field()` in forms, and `esc_*` on every echo.
- Follow WordPress admin UI conventions (`wrap`, `notice`, `widefat`, `form-table`) so the plugin looks native and is accessible.

---

## 10. Data, options, and caching

- Store settings in a **single array option** per feature or group (`plugin_slug_orders_settings`), registered with `register_setting()`, with `autoload` set to false for large or rarely used options.
- Use custom post types/taxonomies/meta for content-like data. Use custom tables only for high-volume or relational data, created and versioned via `dbDelta()` with a stored schema version and an upgrade routine that runs on `plugins_loaded` when the version differs.
- Prefix and register meta with `register_post_meta()` including `type`, `sanitize_callback`, `auth_callback`, and `show_in_rest`.
- Cache expensive work with transients or the object cache (`wp_cache_*`) with explicit expiry and invalidation hooks. Never assume a persistent object cache exists.
- Avoid queries in loops, `posts_per_page => -1` on large sites, `meta_query` on unindexed values at scale, and `orderby => 'rand'`.

---

## 11. Hooks, extensibility, and compatibility

- Expose extension points: `do_action( 'plugin_slug_{feature}_{event}', ... )` and `apply_filters( 'plugin_slug_{feature}_{value}', $value, ... )`. Keep names stable; they are a public API.
- Register hooks in controllers' `register()` methods only. No hooks inside constructors of services or models, and no hook registration at file load time.
- Use the right hook for the job (`init` for CPTs, `rest_api_init` for routes, `admin_menu` for pages, `wp_enqueue_scripts` for assets). Do not do heavy work on `init` or `plugins_loaded` that could be deferred.
- Declare `Requires at least`, `Requires PHP`, `Tested up to`, and (for dependencies) `Requires Plugins` in the header. Check them in the activator.
- Declare compatibility with HPOS, block themes, and multisite where relevant (`before_woocommerce_init` + `FeaturesUtil::declare_compatibility`).
- Never modify core files, other plugins' output through output buffering hacks, or global `$wp_*` state, and never call `flush_rewrite_rules()` on every request.
- Remove only your own hooks. Never call `remove_all_filters()`.

---

## 12. Internationalization and accessibility

- Text domain equals the plugin slug and is a string literal in every call (tools cannot extract variables). Load translations on `init` with `load_plugin_textdomain()` only when supporting WP < 4.6 behavior; WordPress.org-hosted plugins load automatically.
- Use `_n()` for plurals, `_x()` for context, and `sprintf()` with numbered placeholders (`%1$s`) plus a `/* translators: */` comment.
- Admin and frontend output must be keyboard-accessible, labelled (`<label for>`), and use ARIA only when native HTML is insufficient.

---

## 13. Errors, logging, and REST

- Return `\WP_Error` from services and models for expected failures; the controller translates it into a notice, redirect, or `WP_REST_Response` with a proper HTTP status.
- Use REST controllers extending `WP_REST_Controller` with a namespace `plugin-slug/v1`, schema, and versioning. Return consistent shapes.
- Log with a small logger class that writes only when debugging is on; never log credentials, tokens, or personal data.

---

## 14. Tooling, testing, and release

- Run `composer lint` (PHPCS with WPCS, PHPCompatibilityWP), `composer analyse` (PHPStan with the WordPress extension), and the WordPress.org **Plugin Check** plugin before each release.
- Unit-test services with PHPUnit and mocks; integration-test controllers and models with the WP test suite. Services have no WordPress dependencies in their logic where practical, which is why they are easy to test.
- Add GitHub Actions (or equivalent) for lint, analyse, and tests across the supported PHP/WP matrix.
- Build releases from a script that runs `composer install --no-dev -o`, compiles assets, strips dev files (`.git`, `tests`, `phpcs.xml.dist`, `node_modules`, `src` when only `build` is needed), and zips with the correct slug folder.
- Keep `readme.txt` (stable tag, changelog, FAQ, external services disclosure) and `CHANGELOG.md` current; follow semantic versioning; bump version in the plugin header, the constant, and `readme.txt` together.

---

## 15. Workflow when using this skill

1. **Clarify** the plugin name, slug, namespace, text domain, minimum versions, and features.
2. **Scaffold**: bootstrap file, `composer.json`, `phpcs.xml.dist`, `core/` classes, empty `inc/`, `assets/`, `templates/`, `views/` (and `src/` only if needed), `uninstall.php`, `readme.txt`.
3. **Implement one feature at a time**: Model, then Service, then Controller, then register it in `Loader`, then add its template/view and assets.
4. **Self-review each feature** against the checklist below before moving on.
5. **Run or recommend** `composer lint` and `composer analyse`, and explain any intentional `phpcs:ignore` with its reason.

### Pre-delivery checklist

- [ ] Files match the directory structure; no logic in the bootstrap file
- [ ] Class names/files are StudlyCaps and match; PSR-4 namespaces mirror folders; no manual `require` of classes
- [ ] WPCS formatting (tabs, spaces inside parentheses, Yoda conditions, snake_case methods/variables)
- [ ] Every global is prefixed; every string is translatable with the right text domain
- [ ] Every state-changing request checks nonce **and** capability
- [ ] All input is unslashed, validated, and sanitized; all output is escaped late
- [ ] All SQL uses `$wpdb->prepare()` or core APIs
- [ ] REST routes have `permission_callback`, argument schemas, and sanitize/validate callbacks
- [ ] Assets are enqueued conditionally, versioned, and prefixed
- [ ] Activation, deactivation, and uninstall behave correctly (including multisite)
- [ ] Hooks and filters are documented; extension points are stable
- [ ] No `eval`, `extract`, unsafe `unserialize`, hard-coded secrets, or direct `$_REQUEST` use
- [ ] Lint, static analysis, and Plugin Check pass

---

## 16. Output expectations

When asked to build a plugin or feature, deliver:

- The full file tree first (so the user can verify the structure), then each file with its path as a heading.
- Complete, runnable code, not fragments, unless the user asks to modify an existing file (then show only the changed parts with enough context).
- A short note of any decision that deviates from this skill and why (for example, a feature without a Model, or a justified `phpcs:ignore`).
- Setup commands: `composer install`, `composer lint`, and any `npm` build steps.
