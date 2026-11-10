---
name: pr-review
description: Thoroughly review pull requests (PRs) across repositories. Analyzes diffs for security vulnerabilities, logic bugs, regressions, breaking changes, and performance issues. Generates structured audit reports and publishes concise, humanized inline review comments directly to GitHub using exact diff line targeting.
related_skills:
  - human-like-code-engineering
  - security-audit
  - wordpress-security-skill
---

# Pull Request (PR) Review Skill

## Purpose

Perform rigorous, professional, and actionable code reviews on Pull Requests. This skill guides the agent through inspecting PR diffs, catching critical bugs, identifying security and architectural flaws, ensuring backward compatibility, and writing high-quality inline comments that read naturally as if written by an experienced human software engineer.

---

## Review Philosophy & Principles

1. **Human-Engineered Feedback**:
   - Write comments like an experienced senior peer on the team.
   - Be concise, direct, and constructive.
   - **Never sound like generic AI**: Avoid boilerplate intros ("Thanks for the PR!", "Great work on this, however..."), avoid robotic lecturing, and avoid repeating the entire file diff back to the author.
   - Explain **why** the code is problematic and suggest a concrete fix in 1 to 3 sentences.

2. **Accurate Line Targeting**:
   - All inline comments must attach to the exact line in the diff hunk that causes the issue (`RIGHT` side for new/modified lines, `LEFT` side for deleted code).
   - Never comment on unchanged lines outside the diff hunk boundaries.

3. **Multi-Faceted Audit**:
   - **Breaking Changes & Regressions**: Will existing integrations, API consumers, mobile apps, or database records break?
   - **Security Flaws**: IDOR, auth bypasses, CSRF/nonces, XSS, SQLi, insecure deserialization, token lifetime/revocation bugs, missing rate limiting.
   - **Edge Cases & Logic Bugs**: Unhandled nulls, type juggling, missing conditional returns, off-by-one errors, infinite loops.
   - **Performance**: N+1 queries, unindexed queries, memory exhaustion on large datasets.
   - **Standards & Conventions**: Adherence to project conventions (e.g., WPCS for WordPress, PSR-12 for PHP, strict TypeScript/ESLint for JS).

4. **Safety & Permission**:
   - Never publish comments or submit reviews to GitHub unless explicitly instructed by the user. When asked to report first, provide the findings in chat without touching GitHub.

---

## End-to-End Review Workflow

```
┌────────────────────────────────────────────────────────┐
│ 1. PR Discovery & Diff Retrieval                       │
│    Fetch PR metadata, base/head SHAs, full diff hunks  │
└──────────────────────────┬─────────────────────────────┘
                           ▼
┌────────────────────────────────────────────────────────┐
│ 2. Deep Static & Security Audit                        │
│    Analyze changes against codebase & dependencies     │
└──────────────────────────┬─────────────────────────────┘
                           ▼
┌────────────────────────────────────────────────────────┐
│ 3. Findings Synthesis & User Report                    │
│    Categorize issues, exact file/lines, proposed fixes │
└──────────────────────────┬─────────────────────────────┘
                           ▼
┌────────────────────────────────────────────────────────┐
│ 4. Inline Comment Crafting & Publishing (When requested)│
│    Draft short, humanized comments & post atomic review │
└────────────────────────────────────────────────────────┘
```

---

## Step 1: PR Discovery & Diff Retrieval

When given a PR link (e.g., `https://github.com/owner/repo/pull/123`):

1. **Extract Repository & PR Number**:
   - `owner`: repository owner
   - `repo`: repository name
   - `pull_number`: PR ID number

2. **Retrieve PR Diff & Head SHA**:
   - Using GitHub MCP tools (`pull_request_read` with `method: "get"` / `get_diff`) or GitHub API via curl.
   - Save or inspect the full diff to identify changed files and hunk line numbers.
   - Retrieve the latest head commit SHA (`head.sha`) to bind inline comments accurately.

3. **Inspect Prior Review History**:
   - Check existing review comments (`pull_request_read` with `method: "get_review_comments"`) to avoid repeating issues already raised by other reviewers.

---

## Step 2: In-Depth Code Audit Checklist

Review every modified file against the following audit vectors:

### A. Backward Compatibility & Breaking Changes
- Are public function/method signatures modified without default arguments or deprecation layers?
- Are REST API endpoints, query params, or JSON response envelopes altered?
- Did authentication headers or supported auth methods change, breaking external clients?
- Are database schemas or options migrated destructively without backward migration?

### B. Logic & Edge Case Bugs
- Uninitialized variables or array keys accessed without `isset()` / `! empty()`.
- Return values of helper functions (e.g., `false` or `WP_Error`) not checked before proceeding.
- Flawed permission callback logic (e.g., locking out legitimate users from viewing landing pages or public metadata).
- Missing error handling around remote HTTP requests or external system calls.

### C. Security Vulnerabilities
- **Authentication & JWT**:
  - Secret key strength and storage.
  - Stateless token revocation (is `token_version` incremented on logout/password change?).
  - Refresh token rotation and replay prevention.
  - Brute-force / rate-limiting defenses on login routes.
- **Authorization & IDOR**:
  - Are resource IDs validated against `current_user_id` or user capabilities?
  - Can a low-privileged user (Subscriber) modify or access resources belonging to others?
- **Injection & Sanitization**:
  - SQL: raw queries without prepared statements (`$wpdb->prepare()`).
  - XSS: unescaped outputs in templates, `innerHTML` assignments with user-controlled data.
  - External extensions: hardcoded functions like `sodium_bin2base64` without `function_exists()` fallback.

### D. Performance & Scalability
- Database queries inside loops (N+1 query problems).
- Missing LIMIT clauses on large dataset queries.
- Heavy un-cached calculations executed on common hooks.

---

## Step 3: Structuring the Review Report (For the User)

When presenting the review report to the user in chat, follow this format:

```markdown
# Comprehensive Review Report for PR #[number]

**Pull Request:** [Title & Link]
**Branch:** [head] → [base]
**Head Commit:** [commit sha]

---

## Executive Summary
[High-level summary of what the PR does and overall assessment]

---

## 1. Critical Breaking Changes
### 1.1 [Title]
- **File:** `path/to/file.ext`
- **Lines:** line_start–line_end
- **Issue:** [Clear description of what breaks and why]
- **Impact:** [Who/what is affected]
- **Solution:** [Code snippet showing how to fix]

---

## 2. Fatal Errors & Runtime Bugs
[Same structure as above]

---

## 3. Security Vulnerabilities
[Same structure as above]

---

## Summary Matrix of Issues
| # | Severity | File & Lines | Category | Description |
|---|---|---|---|---|
| 1 | Critical | file.php:123 | Breaking Change | ... |
```

---

## Step 4: Crafting & Publishing Inline Review Comments

When the user asks to post review comments on the PR:

### Comment Quality Rules (Humanized Standards)
- **Keep it short**: 1 to 4 sentences maximum.
- **Get straight to the point**: Identify the exact failure mode immediately.
- **Tone**: Professional, friendly peer tone. No AI markers.
- **Include actionable fix**: State or show the intended replacement code succinctly.

#### ❌ Bad (AI-sounding / Bloated)
> "Thank you for submitting this pull request! Upon careful inspection of line 2030, I observed that `sodium_bin2base64` is called directly. In PHP environments where the sodium extension is not installed, this will throw an `Uncaught Error: Call to undefined function`. I strongly recommend that you wrap this in a `function_exists` check or provide a polyfill using standard `base64_encode`. Hope this helps!"

#### ✅ Good (Humanized / Crisp)
> Calling `sodium_bin2base64` directly will cause a fatal error on servers where `ext-sodium` isn't installed or enabled.
> 
> We should check `function_exists( 'sodium_bin2base64' )` and fall back to standard `rtrim( strtr( base64_encode( $data ), '+/', '-_' ), '=' )`.

---

### GitHub API Submission Protocol

To post comments grouped under a single review:

1. **Verify Line Numbers in Diff**:
   - Ensure the comment `line` is an added/modified line (`+` line) on the `RIGHT` side of the diff.
2. **Post as Atomic Review**:
   - Use `POST /repos/{owner}/{repo}/pulls/{pull_number}/reviews` with:
     ```json
     {
       "commit_id": "<HEAD_COMMIT_SHA>",
       "event": "COMMENT",
       "comments": [
         {
           "path": "path/to/file.ext",
           "line": 123,
           "side": "RIGHT",
           "body": "Comment text here."
         }
       ]
     }
     ```
3. **Verify Published Threads**:
   - Confirm review creation and provide direct discussion URLs to the user.

---

## Additional References

- [Inline Comments Guide](./references/inline-comments-guide.md): Examples of humanized vs. AI-sounding review comments.
- [GitHub API Review Reference](./references/github-api-review.md): API endpoints, curl patterns, and token authentication.
- [Review Checklist](./references/review-checklist.md): Detailed checklist covering security, performance, and code quality.
