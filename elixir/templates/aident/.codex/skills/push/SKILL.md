---
name: push
description:
  Submit the current Aident Graphite branch or stack as draft PRs and return
  the current Graphite review link.
---

# Push

## Prerequisites

- `gt` is installed and authenticated.
- `gh` is authenticated if you need PR inspection, checks, or a GitHub PR URL.

## Goals

- Validate the local change before publishing it.
- Submit or update draft PRs through Graphite.
- Return the current review link in Graphite format.

## Steps

1. Confirm the current branch is not `main`.
2. Run the validation appropriate for the scope, following `AGENTS.md`.
   - Use `pnpm`, not `npm`.
   - Do not run `npm run build`.
3. If the clone has not been initialized for Graphite, run
   `gt init --trunk main --no-interactive`.
4. Submit the current stack with `gt stack submit --no-interactive --draft`.
5. If submission fails because the stack is out of date, run the `pull` skill,
   rerun validation, and submit again.
6. Use `gh pr view --json number,url` to confirm the current branch PR when
   GitHub auth is available.
7. When sharing the review link, prefer
   `https://app.graphite.com/github/pr/<number>`.
8. Reply with the PR number plus both the GitHub URL and the Graphite URL when
   available.

## Notes

- Keep PRs as drafts unless a human explicitly asks to publish them.
- If `gt` or `gh` auth is missing, stop and surface the exact blocker instead
  of inventing an alternate publish flow.
