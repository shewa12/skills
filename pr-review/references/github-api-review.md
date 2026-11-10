# GitHub API Pull Request Review Reference

## Overview
This reference outlines how to interact with the GitHub REST API to fetch PR diffs, calculate line coordinates, and post grouped inline review comments.

---

## 1. Authentication

GitHub API requests require a Personal Access Token (`ghp_...` / `github_pat_...`) or OAuth token (`gho_...`) with `repo` scope.

To extract tokens available in the local environment:
```bash
# Check git credential helper
git credential fill <<EOF
protocol=https
host=github.com
EOF

# Or check MCP config
cat ~/.gemini/config/mcp_config.json
```

---

## 2. Retrieving PR Data

### Fetch PR Metadata & Head Commit SHA
```bash
curl -s -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/{owner}/{repo}/pulls/{pull_number} | jq '{head: .head.sha, base: .base.ref, title: .title}'
```

### Fetch Full Diff
```bash
curl -s -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github.v3.diff" \
  https://api.github.com/repos/{owner}/{repo}/pulls/{pull_number} > /tmp/pr.diff
```

---

## 3. Submitting an Atomic Review with Inline Comments

GitHub allows submitting all inline review comments atomically in a single request. This prevents spamming contributors with separate notification emails for every comment.

### API Endpoint:
`POST /repos/{owner}/{repo}/pulls/{pull_number}/reviews`

### Request Body:
```json
{
  "commit_id": "15af0cdcc6f11b9b3ff1a23081231ba5b424836f",
  "event": "COMMENT",
  "body": "Optional top-level review summary comment.",
  "comments": [
    {
      "path": "restapi/RestAuth.php",
      "line": 1136,
      "side": "RIGHT",
      "body": "Inline comment body text."
    },
    {
      "path": "classes/RestAPI.php",
      "line": 275,
      "side": "RIGHT",
      "body": "Another inline comment text."
    }
  ]
}
```

### Parameters:
- `commit_id` (string, required): SHA of the commit being reviewed (must be the latest commit on the PR).
- `event` (string, required): Review action. Use `"COMMENT"` for standard reviews, or `"REQUEST_CHANGES"` if changes are required. If omitted, creates a `PENDING` review.
- `comments[].path` (string, required): Relative file path in the repository.
- `comments[].line` (integer, required): Line number in the new version of the file (`RIGHT` side).
- `comments[].side` (string, optional): `"RIGHT"` for additions/modifications, `"LEFT"` for deletions. Defaults to `"RIGHT"`.
- `comments[].body` (string, required): The comment text in GitHub-flavored markdown.

---

## 4. Line Coordinate Matching Rules

GitHub validates that every comment attaches to a line within an active diff hunk.
If a line is outside the diff, GitHub returns:
```json
{"message": "Validation Failed", "errors": [{"message": "PullRequestReviewComment.line must be part of the diff"}]}
```

### How to Calculate the Correct Line:
1. Parse the diff hunk header: `@@ -old_start,old_count +new_start,new_count @@`.
2. For `side: "RIGHT"`: Start counting from `new_start` for lines starting with `+` or with a space (unchanged context lines inside the hunk).
3. Verify that the target line actually exists in the hunk before making the API request.

---

## 5. Python Automation Script Template

```python
import json
import urllib.request

def submit_review(token, owner, repo, pull_number, commit_sha, comments, event="COMMENT", body=None):
    url = f"https://api.github.com/repos/{owner}/{repo}/pulls/{pull_number}/reviews"
    payload = {
        "commit_id": commit_sha,
        "event": event,
        "comments": comments
    }
    if body:
        payload["body"] = body

    req = urllib.request.Request(
        url,
        data=json.dumps(payload).encode("utf-8"),
        headers={
            "Authorization": f"Bearer {token}",
            "Accept": "application/vnd.github+json",
            "Content-Type": "application/json",
            "User-Agent": "Antigravity-PR-Review"
        },
        method="POST"
    )

    with urllib.request.urlopen(req) as resp:
        return json.loads(resp.read().decode("utf-8"))
```

## 5. Restrictions
1. While reviewing the PRs never switch local development branch, instead checkout for the branches on an isolated area that don't interrupt user's development flow.
