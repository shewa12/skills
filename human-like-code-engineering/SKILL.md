# Human-Like Code Engineering

## Purpose

Write code that is efficient, maintainable, predictable, and easy for a human developer to understand and extend.

AI-generated code must not optimize solely for correctness, brevity, or cleverness. The code should feel like it was written by an experienced human engineer working on a real production codebase.

This skill applies to **all code and configuration**, regardless of language or technology, including but not limited to:

* PHP
* JavaScript
* TypeScript
* React
* HTML
* CSS
* SQL/MySQL
* YAML
* JSON
* Bash/Shell
* Python
* Java
* C/C++
* Dockerfiles
* WordPress
* REST APIs
* Configuration files
* Infrastructure code

---

## Core Principle

> **Write code for the next human developer, not only for the machine.**

Code should be:

1. Correct
2. Simple
3. Readable
4. Consistent
5. Maintainable
6. Extensible
7. Easy to debug
8. Easy to review
9. Consistent with the surrounding codebase

Prefer code that a competent developer can understand quickly without needing to ask the AI why it was written that way.

---

## 1. Prefer Simplicity Over Cleverness

Do not use clever, compressed, or overly abstract solutions when a straightforward implementation is easier to understand.

### Prefer

```php
if ( $user_can_edit ) {
    $this->update_course( $course_id );
}
```

### Avoid

```php
$user_can_edit && $this->update_course( $course_id );
```

The second version may be shorter, but the first communicates intent more clearly.

**Rule:** A few extra lines are acceptable when they significantly improve readability.

---

## 2. Write Code Humans Can Scan

A developer should be able to quickly identify:

* What the code does
* Why it does it
* What data it operates on
* What conditions affect its behavior
* What it returns
* Where side effects occur

Avoid unnecessarily dense expressions.

Prefer:

```javascript
const user = getUser(userId);

if (!user) {
    return null;
}

const courses = getUserCourses(user);

return courses;
```

over:

```javascript
return getUserCourses(getUser(userId)) || null;
```

Do not sacrifice readability merely to reduce the number of lines.

---

## 3. Follow the Existing Codebase

When modifying an existing project, **the existing codebase is the primary style guide**.

Before introducing a new pattern:

* Inspect nearby code.
* Follow existing naming conventions.
* Follow existing file organization.
* Reuse existing utilities.
* Follow existing error-handling patterns.
* Follow existing architectural patterns.
* Follow existing dependency-management conventions.
* Follow the project's formatting and linting rules.

Do not introduce a new abstraction, library, pattern, or coding style unless there is a clear reason.

**Consistency is more valuable than personal preference.**

---

## 4. Do Not Over-Engineer

Do not introduce:

* Unnecessary classes
* Unnecessary interfaces
* Unnecessary abstractions
* Unnecessary design patterns
* Generic utility layers
* Factory layers
* Wrapper classes
* Extra configuration
* Additional dependencies

unless they solve a real problem.

Do not create an abstraction merely because the code *could potentially* need it in the future.

Prefer:

```text
Simple problem → Simple solution
Complex problem → Appropriate abstraction
Repeated problem → Consider abstraction
```

Do not build architecture for hypothetical requirements.

---

## 5. Avoid Premature Optimization

Write clear and correct code first.

Only optimize when:

* There is a known performance problem.
* The requirement explicitly demands performance.
* Profiling or measurable evidence supports the optimization.
* The optimization does not introduce unnecessary complexity.

Do not replace readable code with obscure optimizations without justification.

When performance optimization is necessary, preserve readability as much as reasonably possible.

---

## 6. Use Meaningful Names

Names should communicate intent.

Prefer:

```php
$enrolled_students
$course_id
$subscription_status
$available_courses
```

over:

```php
$x
$data
$temp
$result
$item
$info
```

Avoid meaningless abbreviations unless they are already established conventions in the project.

Prefer:

```javascript
const activeSubscriptions = getSubscriptions();
```

over:

```javascript
const as = getSubscriptions();
```

A good name can eliminate the need for a comment.

---

## 7. Comments Should Explain Why, Not What

Do not write comments that simply translate the code into English.

### Avoid

```php
// Get the user.
$user = get_user_by( 'id', $user_id );
```

### Prefer

```php
// Deleted users are still referenced by historical enrollment records,
// so we keep the record instead of removing the enrollment data.
$user = get_user_by( 'id', $user_id );
```

Use comments when they explain:

* Business rules
* Non-obvious decisions
* Workarounds
* Compatibility requirements
* Security considerations
* Performance decisions
* External constraints
* Unexpected behavior

Do not comment every obvious line.

---

## 8. Preserve Local Context

When changing existing code, do not unnecessarily rewrite surrounding code.

If a function needs one small fix, make one focused change instead of rewriting the entire function.

Avoid:

* Renaming unrelated variables
* Reformatting unrelated code
* Moving unrelated functions
* Changing architecture unnecessarily
* Replacing working code with a preferred style
* Introducing unrelated refactoring

A focused diff is easier to review and safer to merge.

---

## 9. Do Not Repeat Yourself Blindly

DRY is useful, but excessive abstraction can make code harder to understand.

Do not extract code into a helper merely because two pieces look similar.

Before creating an abstraction, determine whether the duplicated code represents the **same concept**.

Prefer understandable duplication over an abstraction that hides important differences.

---

## 10. Keep Functions and Components Focused

A function, method, or component should have a clear responsibility.

Avoid functions that simultaneously:

* Validate data
* Query the database
* Transform data
* Update records
* Send notifications
* Render HTML
* Handle unrelated business rules

When a function becomes difficult to understand, consider separating responsibilities.

However, do not split code into dozens of tiny functions without a meaningful reason.

The goal is **logical separation**, not maximum fragmentation.

---

## 11. Control Nesting

Deep nesting makes code difficult to read.

Prefer early returns and guard clauses where they improve clarity.

Instead of:

```php
if ( $user ) {
    if ( $user->can_edit ) {
        if ( $course ) {
            $this->update_course( $course );
        }
    }
}
```

Prefer:

```php
if ( ! $user ) {
    return;
}

if ( ! $user->can_edit ) {
    return;
}

if ( ! $course ) {
    return;
}

$this->update_course( $course );
```

Do not blindly use early returns everywhere. Choose the structure that makes the control flow easiest to understand.

---

## 12. Make Error Handling Explicit

Do not silently ignore errors unless that behavior is intentional.

Clearly handle:

* Invalid input
* Missing data
* Failed database operations
* API failures
* Permission failures
* Network failures
* Unexpected states

Use the error-handling conventions already established by the project.

Do not catch exceptions simply to suppress them.

Avoid:

```javascript
try {
    doSomething();
} catch (error) {}
```

unless intentionally ignoring the error is part of the design.

---

## 13. Write Defensive Code Without Becoming Excessive

Validate data at appropriate boundaries.

Be particularly careful with:

* User input
* API input
* Database values
* External services
* File operations
* Authentication/authorization
* Serialized or structured data

However, do not add unnecessary defensive checks everywhere.

Use the guarantees provided by the existing architecture.

---

## 14. Security Is Part of Code Quality

Security should never be sacrificed for convenience or brevity.

Consider:

* Input validation
* Output escaping
* SQL parameterization
* Authentication
* Authorization
* CSRF protection
* XSS prevention
* Secure file handling
* Sensitive data exposure
* Proper error messages
* Dependency risks

Use the security mechanisms provided by the framework or platform.

Do not implement custom security mechanisms when established, trusted mechanisms already exist.

---

## 15. Database Code Must Be Readable and Safe

For SQL/MySQL:

* Use meaningful table and column references.
* Parameterize dynamic values.
* Avoid unnecessary queries.
* Avoid `SELECT *` when specific columns are sufficient.
* Make joins understandable.
* Use indexes appropriately.
* Preserve transaction boundaries where required.
* Avoid hiding important database operations inside overly generic helpers.

Prefer readable SQL over unnecessarily compressed queries.

---

## 16. Frontend Code Should Communicate UI Intent

For JavaScript/TypeScript/React:

* Keep component responsibilities clear.
* Use meaningful state names.
* Avoid unnecessary state.
* Avoid unnecessary effects.
* Keep data fetching separate from presentation when appropriate.
* Avoid deeply nested JSX.
* Reuse established project hooks and utilities.
* Follow the project's state-management conventions.
* Do not introduce a state-management library for a small problem.
* Keep event handlers understandable.

Do not turn simple UI behavior into a complex abstraction.

---

## 17. HTML and CSS Should Remain Understandable

For HTML:

* Use semantic elements.
* Keep markup structure logical.
* Avoid unnecessary wrappers.
* Use meaningful attributes.
* Preserve accessibility.

For CSS:

* Follow the existing naming convention.
* Avoid unnecessary specificity.
* Avoid excessive nesting.
* Reuse existing variables/utilities where appropriate.
* Do not introduce complex selectors for simple styling problems.

---

## 18. Configuration Should Be Explicit

For YAML, Docker, CI/CD, and other configuration:

* Prefer explicit configuration over obscure shortcuts.
* Keep related settings together.
* Use meaningful names.
* Avoid unnecessary duplication.
* Do not introduce configuration solely for hypothetical future requirements.
* Preserve the existing project's conventions.

Configuration is code and should be reviewed with the same care as application code.

---

## 19. Avoid AI-Specific Code Smells

Never generate code merely because it looks sophisticated.

Avoid patterns such as:

* Excessive one-liners
* Huge functions
* Generic "helper" functions with unclear responsibilities
* Generic names such as `processData()`
* Unnecessary abstractions
* Excessive comments
* Comments that describe obvious code
* Excessive error handling for impossible states
* Rewriting entire files for a tiny change
* Introducing libraries for trivial functionality
* Copying the same logic into multiple places
* Excessive chaining that hides behavior
* Deeply nested callbacks or conditions
* Overuse of design patterns
* Code that requires understanding several layers to perform a simple operation

If the code looks impressive but is difficult to maintain, it is not good code.

---

## 20. Prefer Familiar Patterns

Use established language and framework conventions.

For example:

* PHP → follow modern PHP and project-specific conventions.
* WordPress → follow WordPress APIs and coding conventions.
* React → follow established React patterns.
* JavaScript → use idiomatic JavaScript.
* SQL → use conventional SQL structure.
* Docker → follow standard Docker practices.
* YAML → maintain clear hierarchical structure.

Do not invent a new pattern when a familiar pattern solves the problem well.

---

## 21. Preserve Backward Compatibility

When working on an existing system, assume existing behavior matters.

Before changing behavior, consider:

* Existing callers
* Existing APIs
* Existing database records
* Existing hooks/events
* Existing integrations
* Existing configuration
* Existing user workflows
* Existing tests

Do not make breaking changes casually.

If a breaking change is required, make it explicit.

---

## 22. Write for Future Modification

Code should be easy to modify without understanding the entire system.

When making a decision, ask:

> "If another developer needs to change this six months from now, will they understand where and how to make the change?"

Prefer localized logic over behavior scattered across many unrelated files.

Keep dependencies and side effects visible.

---

## 23. Match the Scope of the Change

If the request is:

> "Fix this bug."

Do not automatically:

* Refactor the entire module.
* Rename everything.
* Upgrade dependencies.
* Change architecture.
* Rewrite unrelated functions.

If the requested change exposes a genuine architectural problem, explain it separately rather than silently expanding the scope.

---

## 24. Tests Should Be Human-Readable

Tests should communicate expected behavior.

Prefer:

```text
it rejects enrollment when the course is already completed
```

over:

```text
it handles case 4
```

Tests should make business behavior obvious.

When adding tests, cover meaningful behavior rather than writing tests merely to increase coverage numbers.

---

## 25. Explain Important Trade-offs

When there are multiple reasonable implementations, choose the simplest maintainable solution.

If the chosen solution involves a meaningful trade-off, briefly explain it.

For example:

```text
I kept this logic in the existing service instead of introducing a new
repository because this operation is only used in one place and the
additional abstraction would not provide a meaningful benefit.
```

Do not provide lengthy explanations for obvious decisions.

---

## 26. Code Quality Priority

When making implementation decisions, use this general priority:

```text
Correctness
    ↓
Security
    ↓
Maintainability
    ↓
Readability
    ↓
Consistency with the codebase
    ↓
Performance
    ↓
Cleverness / brevity
```

Never sacrifice correctness or security for readability.

Never sacrifice maintainability merely to make code shorter.

---

## 27. Final Human-Review Check

Before presenting generated code, mentally review it as if you were the developer who will maintain it six months later.

Ask:

* Can I understand this without asking the AI?
* Are the names meaningful?
* Is the control flow obvious?
* Is the code unnecessarily clever?
* Did I introduce unnecessary abstractions?
* Did I follow the surrounding codebase?
* Did I modify more than necessary?
* Are comments explaining important decisions rather than obvious operations?
* Are errors handled appropriately?
* Are security concerns addressed?
* Is the implementation easy to test?
* Can another developer extend this without rewriting it?
* Would I be comfortable approving this code in a production pull request?

If the answer to any important question is "no", simplify or improve the implementation before presenting it.

---

## Golden Rule

**Do not write code that merely works. Write code that another experienced developer can understand, review, debug, maintain, and extend without needing the AI that generated it.**

The best AI-generated code should be indistinguishable from thoughtful human-written production code.

