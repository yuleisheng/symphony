---
name: install-symphony
description: Use when a user wants Codex to install or update Symphony locally, verify mise/Elixir/Codex/GitHub/Linear prerequisites, configure a workflow file, and start the local Symphony service. Works for the Aident setup in this repo and for similar forks.
---

# Install Symphony

## Goals

- Get a local Symphony checkout built and runnable.
- Verify the required CLIs and auth.
- Start Symphony with the correct workflow file.
- Validate the first smoke-test issue end to end.

## Inputs To Confirm

- target clone path
- repo or fork URL
- workflow file to use
- required auth and secrets for the target workflow

## Steps

1. Check whether the target repo already exists locally.
   - If it does not, clone it to the requested path.
   - If it does, inspect the current branch and local changes before modifying it.
2. Verify required tools.
   - `mise`
   - `codex`
   - `gh`
   - `gt` when the workflow uses Graphite
   - On macOS, if `mise` is missing and Homebrew is available, install it with `brew install mise`.
   - Surface missing `gh`, `gt`, or `codex` as blockers instead of inventing substitutes.
3. Build Symphony.

```bash
cd <repo>/elixir
mise trust
mise install
mise exec -- mix setup
mise exec -- mix build
```

4. Verify auth and environment.
   - `gh auth status`
   - `codex app-server --help` or confirm Codex auth is already present
   - ensure tracker secrets such as `LINEAR_API_KEY` are available in the same shell that will run Symphony
   - if the workflow uses Graphite, confirm `gt` auth works
5. Choose the workflow file.
   - For the Aident setup in this fork, use `elixir/WORKFLOW.aident.md`.
   - Otherwise use or customize the repo's workflow file for the target project.
6. Start Symphony.

```bash
cd <repo>/elixir
export SYMPHONY_FORK_ROOT=<repo>
mise exec -- ./bin/symphony --i-understand-that-this-will-be-running-without-the-usual-guardrails ./WORKFLOW.aident.md
```

   - Add `--port 4000` if the user wants the dashboard.
7. Validate with a smoke-test issue.
   - move one test issue into an active state
   - confirm a workspace is created under the configured root
   - confirm `.symphony-bootstrap.log` exists
   - confirm the agent creates a branch and reaches the expected PR or handoff state
8. For updates to an existing install:
   - pull the latest repo changes
   - rerun `mise exec -- mix build`
   - restart Symphony

## Aident Notes

- The Aident workflow file in this fork is `elixir/WORKFLOW.aident.md`.
- The current Aident issue states are `Todo`, `In Progress`, `Need Attention`, `PR in Review`, `Rework`, `Merging`, and `Done`.
- The current Aident PR flow keeps PRs in draft mode and requires a concise PR body.

## Notes

- Prefer ephemeral `export` commands unless the user explicitly asks to modify shell rc files.
- Keep setup notes high signal. If blocked, report the missing tool, auth, secret, or status configuration exactly.
