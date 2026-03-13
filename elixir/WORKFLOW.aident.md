---
tracker:
  kind: linear
  project_slug: "aident-ai-f623b0845dc6"
  active_states:
    - Todo
    - In Progress
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
  thread_sandbox: danger-full-access
  turn_sandbox_policy:
    type: dangerFullAccess
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
- Follow `AGENTS.md` in the repo root.
- Workspace bootstrap copied repo-local `commit`, `push`, `pull`, `land`, and
  `linear` skills into `.codex/skills/`.

Core rules:

1. This is an unattended orchestration run. Do not ask a human for follow-up
   unless you are blocked by missing required auth, permissions, or secrets.
2. Use `pnpm`, never `npm`.
3. Prefer Graphite (`gt`) over raw Git for branch, commit, submit, and merge
   flows.
4. Keep changes tightly scoped to the ticket.
5. Keep the final Linear update high-signal. Post a single final workpad update
   before handoff, or earlier only if blocked.
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
2. Do not post routine progress updates to Linear. Update the workpad only once
   at final handoff to `Human Review`, or earlier only if blocked.
3. Keep local notes as needed, but only include high-signal details in the
   final Linear update.
4. If the current branch is `main` or HEAD is detached, create a dedicated
   working branch before any edits:
   - Prefer the Linear `branch_name` when it is present and sane.
   - Otherwise derive a concise branch name from `{{ issue.identifier }}` and
     the ticket title.
   - Use `gt branch create <branch-name>`.
5. If `.symphony-bootstrap.log` contains setup failures, account for them in
   the plan before implementation.

## Step 2: Execute the work

1. Implement the smallest change that satisfies the ticket.
2. Follow repo instructions from `AGENTS.md`, including:
   - strong typing,
   - fail-fast behavior,
   - minimal comments,
   - detailed test plan expectations.
3. Run the relevant validation for the scope and keep only the key E2E
   verification plan for the final handoff unless the ticket explicitly
   requires more detail.
4. If the ticket or comments contain a required `Validation`, `Test Plan`, or
   `Testing` section, complete every item and include the results in the final
   handoff.
5. Before any publish step, confirm the local validation is green.
6. Use the `commit` skill to create logical commits when the change is ready.
7. Use the `push` skill to submit or update the draft PR.
8. Attach the GitHub PR URL to the Linear issue using the `linear` skill.
9. When work is complete, update the workpad once with a brief high-signal
   handoff:
   - if blocked: briefly describe the blocker, why it blocks completion, and
     the exact action needed to unblock;
   - otherwise: briefly describe the issue, the implemented solution, and the
     E2E verification plan;
   - include PR links when available.
10. Only then move the issue to `Human Review`.

## Step 3: Human review handling

1. In `Human Review`, do not start new feature work.
2. Poll for PR feedback, CI status, and approval updates.
3. If feedback requires code changes, move the issue to `Rework`.
4. If a human moves the issue to `Merging`, open and follow
   `.codex/skills/land/SKILL.md`.

## Step 4: Rework handling

1. Re-read the full issue, the workpad, and all review comments.
2. Do not post routine progress updates during rework. Update the workpad again
   only at the end of the rework pass, or earlier only if blocked.
3. Continue the normal execution flow: implement, validate, `commit`, `push`,
   and return the issue to `Human Review` only after all requested feedback is
   addressed.

## Blocked-access escape hatch

Use this only for true external blockers after exhausting normal repo-local
options.

- Valid blockers include missing `codex` auth, missing Linear auth, missing
  Graphite auth, missing GitHub auth, or missing secrets required by the task.
- If blocked, update the workpad briefly with:
  - the blocker,
  - why it blocks completion,
  - the exact action needed to unblock.
- Then move the issue to `Human Review`.
