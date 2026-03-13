---
tracker:
  kind: linear
  project_slug: "aident-ai-f623b0845dc6"
  active_states:
    - Todo
    - In Progress
    - Human Review
    - Rework
    - Merging
  terminal_states:
    - Closed
    - Cancelled
    - Canceled
    - Duplicate
    - Done
polling:
  interval_ms: 5000
workspace:
  root: ~/code/symphony-workspaces/aident
hooks:
  after_create: |
    sh "${SYMPHONY_FORK_ROOT:?set SYMPHONY_FORK_ROOT}/elixir/scripts/bootstrap_aident_workspace.sh"
  timeout_ms: 1800000
agent:
  max_concurrent_agents: 10
  max_turns: 20
codex:
  command: codex --config shell_environment_policy.inherit=all --config model_reasoning_effort=xhigh --model gpt-5.3-codex app-server
  approval_policy: never
  thread_sandbox: workspace-write
  turn_sandbox_policy:
    type: workspaceWrite
---

You are working on an Aident Linear ticket `{{ issue.identifier }}`.

{% if attempt %}
Continuation context:

- This is retry attempt #{{ attempt }} because the ticket is still active.
- Resume from the current workspace state instead of restarting from scratch.
- Do not redo completed investigation or validation unless the code changed.
{% endif %}

Issue context:
Identifier: {{ issue.identifier }}
Title: {{ issue.title }}
Current status: {{ issue.state }}
Labels: {{ issue.labels }}
URL: {{ issue.url }}

Suggested branch from Linear:
{% if issue.branch_name %}
{{ issue.branch_name }}
{% else %}
No Linear branch name provided.
{% endif %}

Description:
{% if issue.description %}
{{ issue.description }}
{% else %}
No description provided.
{% endif %}

Bootstrap context:

- Read `.symphony-bootstrap.log` before making changes.
- Follow `AGENTS.md` and `CLAUDE.md` in the repo root.
- Workspace bootstrap copied repo-local `commit`, `push`, `pull`, `land`, and
  `linear` skills into `.codex/skills/`.

Core rules:

1. This is an unattended orchestration run. Do not ask a human for follow-up
   unless you are blocked by missing required auth, permissions, or secrets.
2. Use `pnpm`, never `npm`.
3. Prefer Graphite (`gt`) over raw Git for branch, commit, submit, and merge
   flows.
4. Keep changes tightly scoped to the ticket.
5. Provide a detailed test plan and validation evidence in the workpad before
   handoff.
6. Work only in the provided workspace copy of `aident.ai`.

## Related skills

- `linear`: use Symphony's `linear_graphql` tool for raw Linear GraphQL work.
- `pull`: sync the current branch or stack with latest `origin/main`.
- `commit`: create Graphite-managed commits that match Aident's commit rules.
- `push`: submit or update draft Graphite PRs.
- `land`: merge an approved PR with Graphite when the ticket reaches `Merging`.

## Status map

- `Backlog` -> out of scope for this workflow; do not modify.
- `Todo` -> immediately move to `In Progress`, then start execution.
- `In Progress` -> active implementation.
- `Human Review` -> work is ready and waiting on human review.
- `Rework` -> review feedback requires another implementation pass.
- `Merging` -> approved and ready to land through the `land` skill.
- `Done` -> terminal; no work required.

## Step 0: Route by current issue state

1. Fetch the issue by identifier and confirm the current state.
2. Route accordingly:
   - `Backlog` -> do nothing.
   - `Todo` -> move to `In Progress`, then begin execution.
   - `In Progress` -> continue execution from the existing workpad.
   - `Human Review` -> wait for new review feedback or approval.
   - `Rework` -> begin a fresh execution pass focused on review feedback.
   - `Merging` -> open and follow `.codex/skills/land/SKILL.md`.
   - `Done` -> do nothing.

## Step 1: Start or continue execution

1. Find or create a single persistent workpad comment headed
   `## Codex Workpad`.
2. Update the workpad before new code changes:
   - current plan,
   - acceptance criteria,
   - validation checklist,
   - blockers and assumptions.
3. Add a compact environment stamp in the workpad using the format
   `<host>:<abs-workdir>@<short-sha>`.
4. Reproduce the issue or capture the current baseline behavior before editing
   code. Record the reproduction evidence in the workpad.
5. Run the `pull` skill before editing code and record the resulting `HEAD` SHA
   in the workpad.
6. If the current branch is `main` or HEAD is detached, create a dedicated
   working branch before any edits:
   - Prefer the Linear `branch_name` when it is present and sane.
   - Otherwise derive a concise branch name from `{{ issue.identifier }}` and
     the ticket title.
   - Use `gt branch create <branch-name>`.
7. If `.symphony-bootstrap.log` contains setup failures, account for them in
   the plan before implementation.

## Step 2: Execute the work

1. Follow the workpad plan and keep it current after each meaningful milestone.
2. Implement the smallest change that satisfies the ticket.
3. Follow repo instructions from `AGENTS.md` and `CLAUDE.md`, including:
   - strong typing,
   - fail-fast behavior,
   - minimal comments,
   - detailed test plan expectations.
4. Run the relevant validation for the scope and record exact commands plus
   results in the workpad.
5. If the ticket or comments contain a required `Validation`, `Test Plan`, or
   `Testing` section, copy it into the workpad and complete every item.
6. Before any publish step, confirm the local validation is green.
7. Use the `commit` skill to create logical commits when the change is ready.
8. Use the `push` skill to submit or update the draft PR.
9. Attach the GitHub PR URL to the Linear issue using the `linear` skill.
10. When work is complete, refresh the workpad so plan, acceptance criteria,
    and validation all match reality.
11. Only then move the issue to `Human Review`.

## Step 3: Human review handling

1. In `Human Review`, do not start new feature work.
2. Poll for PR feedback, CI status, and approval updates.
3. If feedback requires code changes, move the issue to `Rework`.
4. If a human moves the issue to `Merging`, open and follow
   `.codex/skills/land/SKILL.md`.

## Step 4: Rework handling

1. Re-read the full issue, the workpad, and all review comments.
2. Update the workpad with a fresh plan focused on the requested changes.
3. Continue the normal execution flow: implement, validate, `commit`, `push`,
   and return the issue to `Human Review` only after all requested feedback is
   addressed.

## Blocked-access escape hatch

Use this only for true external blockers after exhausting normal repo-local
options.

- Valid blockers include missing `codex` auth, missing Linear auth, missing
  Graphite auth, missing GitHub auth, or missing secrets required by the task.
- If blocked, update the workpad with:
  - what is missing,
  - why it blocks completion,
  - the exact action needed to unblock.
- Then move the issue to `Human Review`.
