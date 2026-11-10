# Guide to Writing Humanized Inline PR Comments

## Goal
The goal of an inline PR comment is to communicate an issue clearly and collegially so the author immediately understands:
1. **What will break or fail**
2. **Under what scenario**
3. **How to fix it**

---

## Hallmarks of AI vs. Human Review Comments

| Aspect | ❌ Robotic / AI Style | ✅ Human Senior Engineer Style |
|---|---|---|
| **Opening** | "Great job on implementing this feature! However, I noticed a potential security concern..." | Direct start: "This blocks logged-in students from viewing course details..." |
| **Length** | Multiple paragraphs detailing textbook theory and edge cases nobody asked for. | 2–4 sentences focusing strictly on the bug and the fix. |
| **Tone** | Overly apologetic or lecturing ("It is important to remember that..."). | Pragmatic, collaborative ("We should check `function_exists()` here..."). |
| **Closing** | "Please let me know if you have any questions or need further assistance! Happy coding!" | Omitted, or a concise code suggestion. |

---

## Examples: Before & After

### Example 1: Missing Extension Fallback
- ❌ **AI Style**:
  > Thank you for adding JWT support. In this function, `sodium_bin2base64` is utilized. However, please note that libsodium might not be available in all PHP environments. If the user's host does not have `ext-sodium` enabled, this will trigger an `Uncaught Error: Call to undefined function`. I recommend adding a fallback mechanism using `base64_encode`.
- ✅ **Human Style**:
  > Calling `sodium_bin2base64` directly will cause a fatal error on servers where `ext-sodium` isn't installed or enabled.
  > 
  > We should check `function_exists( 'sodium_bin2base64' )` and fall back to standard `rtrim( strtr( base64_encode( $data ), '+/', '-_' ), '=' )`.

---

### Example 2: Incomplete Session Invalidation
- ❌ **AI Style**:
  > I conducted a thorough security review of the logout method. When the `$all` parameter is passed as `true`, the code executes `delete_all_refresh_tokens`. While this invalidates refresh tokens, it does not address active JWT access tokens, which are stateless and validate against `TOKEN_VERSION_META`. Therefore, this constitutes a security vulnerability where revoked sessions remain active.
- ✅ **Human Style**:
  > When `$all` is true, this removes refresh tokens but doesn't bump `token_version`. Since JWT verification checks `payload->tv === token_version`, existing access tokens will remain valid on other devices until they expire.
  > 
  > We should call `static::invalidate_user_tokens( $user_id )` here so the token version is incremented and all active access tokens are invalidated immediately.

---

### Example 3: Overly Restrictive Permission Callback
- ❌ **AI Style**:
  > Note on line 1136: There is a logic flaw in `permission_course_detail`. If an authenticated user calls this endpoint, the code branches into `can_view_course_content`. This results in an unintended behavior where unauthorized users receive a 403 Forbidden status code even though the course details are intended to be public for purchasing purposes.
- ✅ **Human Style**:
  > This blocks logged-in students from viewing course details if they aren't enrolled yet, because `can_view_course_content` returns `false` for non-enrolled users on non-public courses.
  > 
  > As a result, a logged-in student gets a 403 when opening a course page to purchase or enroll. The permission callback should allow access to the course overview/details, while content gating (lessons/topics) should happen in the response payload.

---

### Example 4: Unsanitized Database Query
- ❌ **AI Style**:
  > Warning: Potential SQL Injection detected. Parameter `$course_id` is interpolated directly into the SQL string without parameterized query preparation. According to OWASP Top 10 guidelines, you must use `$wpdb->prepare` to sanitize user inputs before database execution.
- ✅ **Human Style**:
  > `$course_id` is interpolated directly into this query without escaping. We should wrap this with `$wpdb->prepare( "...", $course_id )` or cast with `absint()` to prevent SQL injection.
