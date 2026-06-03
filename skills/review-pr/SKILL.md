---
name: review-pr
description: Use to review an open pull request against the Constitution and the linked Issue's acceptance criteria — checks single-concern scope, correct test levels, all UX states, and no secrets, then approves or requests changes without merging.
---

# review-pr

Review one open PR the way a senior engineer would: against the **Constitution** and
the **linked Issue's acceptance criteria**. Approve or request changes — but never
merge without an explicit human go-ahead.

```
OPEN PR → review vs Constitution + Issue → approve / request changes → human merges
```

## When to use
- A PR is open and awaiting review.
- You want a disciplined gate before a human merges.

## Before you start
Read `CONSTITUTION.md` (Prime directives, Definition of Done, Testing pyramid, UX
states) and `CLAUDE.md` / `AGENTS.md`. Pull the PR and its linked Issue:
```sh
gh pr view <pr> --json title,body,files,additions,deletions
gh pr diff <pr>
gh issue view <issue>     # the one the PR Closes
```

## Steps

1. **Map the PR to its Issue.** Find the `Closes #<n>` link. Read the Issue's
   acceptance criteria — they are the bar this PR must clear. A PR with no linked
   Issue is a finding: request the link.

2. **Check scope.** It must be **one concern**. If the diff does more than one thing
   (can't be described in a sentence without "and"), request a split.

3. **Check the tests.** Every behavior change ships a test at the right level
   (unit / integration / e2e per the pyramid). Verify the test would **fail without
   the change** (not a tautology), and that integration points aren't mocked where
   they could cheaply run for real. Missing or vacuous tests block approval.

4. **Check the UX states.** If the PR touches a UI surface, confirm
   **loading / not-connected (unauthorized) / empty / error** are each handled
   distinctly — not just the happy "loaded" path.

5. **Check for secrets.** Scan the diff for any credential, token, key, DSN, or
   publisher id committed to code/config/CI. A committed secret is a stop-the-line
   finding — request changes immediately.

6. **Check the Definition of Done.** Walk the `CONSTITUTION.md` checklist:
   acceptance criteria met, small/single-concern, tests present, lint/typecheck/test
   green in CI, UX states handled, no secrets, docs/templates updated if affected,
   PR closes its Issue.

7. **Leave the verdict** with specific, actionable comments:
   ```sh
   # request changes:
   gh pr review <pr> --request-changes --body "<numbered, specific findings>"
   # or approve when the bar is met:
   gh pr review <pr> --approve --body "<what you verified>"
   ```

8. **Report.** State approve vs request-changes and the reasons. If approved, say it
   is ready for a **human** to merge — do not merge it yourself.

## Guardrails
- **Never merge without explicit human approval**, even on a green, approved PR.
- Findings must be specific and actionable — point at the line and say what to do.
- A secret in the diff, a missing test for a behavior change, or an unhandled UX
  state each block approval on their own.
