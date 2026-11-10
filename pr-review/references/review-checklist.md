# Comprehensive PR Review Checklist

Use this checklist during PR analysis to guarantee all critical vulnerability classes, regressions, and design flaws are audited.

---

## 1. Security & Authentication Audit

- [ ] **Authentication Flow**:
  - Are credentials transported securely?
  - Are tokens generated with cryptographically secure random bytes?
  - Does the system enforce a strong signing algorithm (e.g., HS256 with long secret, RS256)?
  - Are tokens bound to an expiration timestamp (`exp`) and not accepted indefinitely?
- [ ] **Revocation & Invalidation**:
  - Does logging out invalidate both refresh and access tokens?
  - If access tokens are stateless JWTs, is there a server-side version (`token_version`) or blocklist checked on verification?
  - Are tokens invalidated upon password reset or security-critical profile changes?
- [ ] **Authorization & Permissions (IDOR)**:
  - Is every resource access gated by user ownership or explicit capability checks?
  - Are IDs passed via request parameters (URL, query, body) verified against the authenticated user (`get_current_user_id()`)?
  - Do permission callbacks return boolean or `WP_Error` reliably?
- [ ] **Input Sanitization & Output Escaping**:
  - Are inputs sanitized at ingress using appropriate sanitizers (`sanitize_text_field`, `absint`, `wp_kses`)?
  - Are outputs escaped at egress before rendering (`esc_html`, `esc_attr`, `esc_url`)?
  - In JavaScript, are dynamic values rendered via `textContent` rather than `innerHTML`?
- [ ] **Database & Query Safety**:
  - Are all SQL queries constructed using `$wpdb->prepare()` with proper placeholders (`%d`, `%s`)?
  - Are table and column names safely whitelisted or escaped with backticks?
- [ ] **Rate Limiting & Abuse**:
  - Are authentication, password reset, and resource-heavy endpoints throttled?

---

## 2. Backward Compatibility & Regressions

- [ ] **Public APIs & Contracts**:
  - Have any existing endpoint paths or HTTP methods changed?
  - Are response JSON structures backwards-compatible with existing mobile apps and third-party consumers?
  - Are required fields still present in payloads?
  - Do status codes match expected contracts (e.g., avoiding switching from 200 to 404 abruptly if clients expect empty arrays)?
- [ ] **Function & Method Signatures**:
  - Were parameters added to existing public methods without default values?
  - Were method visibility levels reduced (e.g., `public` to `private`)?
  - Were classes, traits, or functions renamed or moved without aliases?
- [ ] **Database Migrations**:
  - Do schema migrations handle upgrades and downgrades cleanly?
  - Are existing rows with null or default values properly supported?

---

## 3. Logic & Runtime Robustness

- [ ] **Dependency & Extension Checks**:
  - Does the code call external PHP extensions (e.g., `sodium`, `curl`, `mbstring`, `intl`) without checking `function_exists()` or `extension_loaded()`?
- [ ] **Null & Type Safety**:
  - Are array indices checked before access (`isset()`, `! empty()`, or null coalescing `??`)?
  - Are objects verified before method invocations (avoiding `Call to a member function on null`)?
  - Does strict comparison (`===`, `in_array(..., true)`) account for type conversions (e.g., strings vs integers from database or query params)?
- [ ] **Edge Cases**:
  - Does the code handle empty lists, zero values, negative numbers, and boundary limits gracefully?
  - Are transient or cached lookups refreshed on underlying data mutation?

---

## 4. Performance & Scalability

- [ ] **Query Efficiency**:
  - Are queries executed in loops (N+1 query issue)?
  - Are joins indexed and query execution plans optimized?
  - Are queries paginated with `LIMIT` and `OFFSET` rather than pulling thousands of rows into PHP memory?
- [ ] **Memory & Caching**:
  - Are large payloads streamed or chunked?
  - Is transient caching applied to expensive external HTTP requests or heavy computations?
