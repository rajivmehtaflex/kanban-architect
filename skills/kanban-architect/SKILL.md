---
name: kanban-architect
description: Use when turning PRDs/specs/high-level requirements into Hermes Kanban boards, atomic task DAGs, profile-routed worker cards, and review-gated autonomous execution plans.
version: 2.0.0
author: Hermes Agent + Rajiv Mehta
license: MIT
metadata:
  hermes:
    tags: [kanban, orchestration, decomposition, multi-agent, prd, planning, tui]
    related_skills: [kanban-orchestrator, kanban-worker, hermes-agent]
---

# Kanban Architect

## Overview

Use this skill to transform a high-level requirement, PRD, spec, or TUI request into a durable Hermes Kanban workflow. The goal is not merely to create a few broad cards; the goal is to create a dependency-aware task graph that workers can execute with low ambiguity, good observability, and clean recovery paths.

This skill is for **Level 3 autonomous orchestration**:

```text
Requirement / PRD
  -> preflight
  -> triage card
  -> atomic DAG proposal
  -> human approval if complex
  -> Kanban cards with parents and assignees
  -> gateway dispatcher execution
  -> monitoring, recovery, review, root completion
```

The most important rule: **decompose into executable increments, not vague modules**. A card like `Implement backend` is usually too large. Prefer cards like `Implement GET /todos endpoint`, `Add SQLite todo repository`, or `Verify Vite proxy against FastAPI`.

## When to Use

Use this skill when the user asks to:

- Build a product, app, feature, or workflow using Hermes Kanban.
- Convert a `prd.md`, `spec.md`, issue, or natural-language goal into Kanban cards.
- Use Hermes TUI with a skill to create a project board and run workers.
- Decompose work across multiple profiles such as `backend-dev`, `frontend-dev`, `g-proc`, `qa`, or `reviewer`.
- Diagnose why a Kanban run produced too-coarse cards or worker hurdles.
- Create a reliable multi-agent DAG with dependencies, review gates, and monitoring.

Do **not** use Kanban for a tiny one-shot question or a simple command that one agent can finish immediately. Use direct execution or `delegate_task` for short subtasks that do not need persistence, profile routing, recovery, or an audit trail.

## Mental Model

### Board

A board is the durable project/workstream container. It stores cards, dependencies, comments, events, runs, and worker status. Use one board per project or major workstream.

### Profile

A profile is a named worker persona with its own config, model/provider, tools, skills, and environment. Cards are assigned to profiles. Do not invent assignee names; discover available profiles first.

### Task / Card

A card is a concrete unit of work. A good card has:

- One clear goal.
- One best-fit assignee.
- Explicit parent dependencies.
- Clear scope and out-of-scope boundaries.
- A done condition.
- A verification method.
- A handoff expectation.

### DAG

The task graph is a Directed Acyclic Graph. Parent tasks must complete before child tasks become ready. Independent tasks should be parallel; dependent tasks must use parent links instead of prose like “wait for X”.

## Phase 0: Preflight Before Creating Cards

Run preflight before decomposing any serious project. Most Kanban hurdles come from missing auth, wrong profile config, broken model provider, or vague requirements.

### Required checks

```bash
hermes profile list
hermes kanban assignees
hermes doctor
hermes gateway status
```

If GitHub/repo work is involved:

```bash
gh auth status
gh api user --jq '.login'
```

If model/provider failures happened recently, run a tiny smoke test in the intended profile or verify config:

```bash
hermes -p <profile> chat -q "Reply with OK if this profile can call its configured model." -Q
```

### Preflight questions

Ask or infer:

1. Which board/project slug should this use?
2. Which profiles exist and what does each do?
3. Which decomposition mode should be used: module, atomic, review-gated, swarm, or milestone?
4. Is GitHub access needed? If yes, is `gh` or `GH_TOKEN` visible to workers?
5. Should coding tasks use `worktree`, `dir:<path>`, or scratch workspace?
6. Does the user want graph approval before card creation? For complex work, default to yes.

## Decomposition Modes

Choose the mode deliberately.

| Mode | Use when | Shape |
|---|---|---|
| `module` | User wants a broad high-level board | backend / frontend / docs / review |
| `atomic` | User wants reliable execution and observability | endpoint-by-endpoint, component-by-component |
| `review-gated` | Code quality matters or user wants human-in-loop | implementation -> review -> fix loops |
| `swarm` | Many independent lanes can run in parallel | parallel workers -> verifier -> synthesizer |
| `milestone` | Project needs staged delivery | MVP -> tests -> polish -> release |

Default to **atomic** for implementation projects unless the user explicitly asks for a coarse board.

## Atomic Task Rules

A task is too large if it:

- Touches more than 2-3 unrelated files.
- Implements multiple endpoints, screens, or features.
- Mixes backend and frontend work.
- Mixes implementation and review.
- Requires both environment setup and product logic.
- Cannot be verified with one clear command or inspection.
- Would likely take more than 3-5 tool calls for an already-oriented worker.
- Has a vague title like `Implement backend`, `Build frontend`, `Set up everything`, or `Fix app`.

A task is good if it:

- Has one output.
- Has one owner.
- Names exact scope.
- Has parent dependencies instead of prose coordination.
- Has a done condition.
- Can be retried independently.

### Bad vs good

Bad:

```text
Implement FastAPI backend with SQLite and uv
```

Good:

```text
Implement GET /todos endpoint
Assignee: backend-dev
Parents: SQLite repository task
Scope: add only read/list route
Out of scope: create/update/delete routes
Done: GET /todos returns 200 and JSON array
Verify: uv run python smoke_get_todos.py or curl localhost:8000/todos
```

## Graph-Before-Create Rule

For any multi-card project, propose the DAG before creating cards unless the user says “auto-create”. Show:

- Task ID placeholder, title, assignee.
- Parent dependencies.
- Scope.
- Done condition.
- Verification method.

Then ask for approval or edits.

Example response shape:

```text
Proposed DAG for board `g-todo`:

T1 [default] Write OpenAPI + data model spec
  parents: none
  done: spec covers CRUD routes and Todo fields

T2 [g-proc] Initialize repo and workspace
  parents: none
  done: repo exists, uv backend and Vite frontend dirs exist

T3 [backend-dev] Create FastAPI app skeleton
  parents: T1,T2
  done: app starts and /health returns 200

...

Approve this graph, or choose finer/coarser granularity?
```

## Task Body Template

Every created card should have a body like this:

```md
## Goal
<one concrete outcome>

## Scope
- <included item>
- <included item>

## Out of Scope
- <explicitly excluded item>

## Inputs
- Parent tasks: <ids or titles>
- Files/specs: <paths>

## Implementation Notes
- <profile-specific guidance>

## Done Condition
- <observable completion criteria>

## Verification
- Command: `<command>`
- Expected: <expected result>

## Handoff
On completion, summarize changed files, tests/verification run, decisions, and any follow-up cards created.
```

## Operational Workflow

### Phase 1: Requirement ingestion / triage

1. Read the PRD/spec/request.
2. Extract a concise title under 80 characters.
3. Create the root card in triage.

```bash
hermes kanban create "<title>" --triage --body "$(cat prd.md)"
```

Capture the root task id.

### Phase 2: Profile-aware planning

Discover real profiles:

```bash
hermes profile list
hermes kanban assignees
```

Map lanes only to real profiles. If no profile fits a lane, ask the user whether to create a profile or assign to an existing one. Do not invent assignees such as `researcher` or `qa-dev` unless they exist.

### Phase 3: Propose DAG

Draft the atomic DAG. Prefer functional increments over module blobs.

Parallelize independent lanes. Link only true dependencies. Words like “also” or “finally” do not automatically imply dependencies.

### Phase 4: Create cards

After approval, either use `hermes kanban decompose <root_id>` if the built-in decomposer is sufficient, or manually create stricter cards.

Manual creation example:

```bash
hermes kanban create "Implement GET /todos endpoint" \
  --assignee backend-dev \
  --parent t_sqlite_repo \
  --body "<task body from template>"
```

If the child depends on unfinished parents, include parent links when creating it. Do not create all tasks as ready cards and link later; a dispatcher can claim them before dependencies exist.

### Phase 5: Dispatch and monitor

Start or verify gateway:

```bash
hermes gateway start
```

Monitor with:

```bash
hermes kanban watch
hermes kanban list
hermes kanban runs <task_id>
hermes kanban log <task_id>
hermes kanban diagnostics
```

Use `watch` for live events, `runs` for retry history, `log` for deep debugging, and `diagnostics` for blocked or failed worker issues.

### Phase 6: Review and root completion

Create explicit review/verification cards. The root task should not be considered done until:

- Implementation tasks are done.
- Verification tasks are done.
- Review/handoff is complete.
- Any required human approval has happened.

## Common DAG Templates

### Full-stack app

```text
Root project
├── Write product/API/data model spec                  [default]
├── Initialize repo/project structure                  [g-proc]
├── Create backend scaffold                            [backend-dev] depends spec, repo
├── Implement database/repository layer                [backend-dev] depends backend scaffold
├── Implement read endpoints                           [backend-dev] depends repository
├── Implement mutation endpoints                       [backend-dev] depends repository
├── Backend smoke tests                                [backend-dev] depends endpoints
├── Create frontend scaffold                           [frontend-dev] depends spec, repo
├── Create frontend API client                         [frontend-dev] depends backend tests, frontend scaffold
├── Implement read/list UI                             [frontend-dev] depends API client
├── Implement mutation UI                              [frontend-dev] depends API client
├── Frontend styling/loading/error states              [frontend-dev] depends UI tasks
├── End-to-end local verification                      [qa/default] depends backend + frontend
└── Final root review                                  [default] depends verification
```

### Bugfix

```text
Root bug
├── Reproduce bug                                      [qa/default]
├── Identify root cause                                [engineer] depends reproduce
├── Add regression test                                [engineer/qa] depends root cause
├── Implement minimal fix                              [engineer] depends root cause
├── Verify regression                                  [qa/default] depends test + fix
└── Review and close                                   [reviewer/default] depends verify
```

### Research + implementation

```text
Root goal
├── Research docs/options                              [research/default]
├── Inspect codebase constraints                       [engineer]
├── Synthesize implementation plan                     [default] depends research + inspection
├── Implement selected approach                        [engineer] depends plan
├── Verify/benchmark                                   [qa/default] depends implementation
└── Final recommendation / handoff                     [default] depends verify
```

### Migration

```text
Root migration
├── Inventory current behavior                         [engineer/default]
├── Define compatibility contract                      [default] depends inventory
├── Build new scaffold                                 [engineer] depends contract
├── Port one functional slice                          [engineer] depends scaffold
├── Add parity tests                                   [qa/engineer] depends slice
├── Repeat slices or spawn sub-DAGs                    [engineer] depends parity pattern
└── Cutover review                                     [reviewer/default] depends all slices
```

## Full-Stack Todo Example: Better Than Module Blobs

Avoid this coarse graph:

```text
Spec
Project structure
Backend
Frontend
Root
```

Prefer this atomic graph:

```text
Root: g-todo app
├── T1: Write OpenAPI + data model spec                 [default]
├── T2: Initialize repo, uv backend, Vite frontend dirs  [g-proc]
├── T3: Create FastAPI app skeleton                      [backend-dev] depends T1,T2
├── T4: Implement SQLite todo repository                 [backend-dev] depends T3
├── T5: Implement GET /todos                             [backend-dev] depends T4
├── T6: Implement POST /todos                            [backend-dev] depends T4
├── T7: Implement PATCH/PUT /todos/{id}                  [backend-dev] depends T4
├── T8: Implement DELETE /todos/{id}                     [backend-dev] depends T4
├── T9: Add backend smoke tests                          [backend-dev] depends T5,T6,T7,T8
├── T10: Create Vite React scaffold                      [frontend-dev] depends T1,T2
├── T11: Create frontend API client                      [frontend-dev] depends T9,T10
├── T12: Implement TodoList read UI                      [frontend-dev] depends T11
├── T13: Implement add todo UI                           [frontend-dev] depends T11
├── T14: Implement toggle/delete UI                      [frontend-dev] depends T11
├── T15: Add frontend styling/loading/error states       [frontend-dev] depends T12,T13,T14
├── T16: End-to-end local verification                   [qa/default] depends T9,T15
└── T17: Final root review and completion                [default] depends T16
```

## TUI Usage Pattern

In Hermes TUI, the skill is procedural knowledge, not a slash command. Load or reference it naturally:

```text
Use the kanban-architect skill.
Read ./prd.md and propose an atomic Kanban DAG first.
Do not create cards until I approve.
Use only profiles from `hermes profile list`.
```

After approval:

```text
Create the approved Kanban cards and start/monitor the dispatcher.
```

If a run gets stuck:

```text
Use kanban-architect recovery. Inspect `hermes kanban diagnostics`, `runs`, and logs; propose whether to reclaim, reassign, unblock, or fix profile config.
```

## Recovery Playbook

### Worker auth failure

Symptoms: `gh` works in host session but not in worker.

Actions:

1. Check worker profile env/config.
2. Prefer `GH_TOKEN` in accessible env when CLI auth is not inherited.
3. Move repo/bootstrap work to a `g-proc` profile or host-driven setup card.
4. Reclaim or reassign the failed task after fixing auth.

### Provider/model failure

Symptoms: HTTP 500, 404 model not found, unknown provider.

Actions:

1. Run `hermes doctor`.
2. Check main and profile-specific `config.yaml`.
3. Verify profile model smoke test.
4. Reclaim stuck tasks after updating provider config.
5. Consider separate models for planning, coding, and review.

### Coarse decomposition

Symptoms: tasks like `Implement backend` or `Build frontend` run for a long time with opaque progress.

Actions:

1. Stop and propose an atomic DAG.
2. Split large cards into child cards by endpoint/component/test.
3. Use explicit parent links.
4. Add verification/review cards.

### Worker exits without completion

Symptoms: task gets auto-blocked or remains running stale.

Actions:

```bash
hermes kanban runs <task_id>
hermes kanban log <task_id>
hermes kanban reclaim <task_id>
# or
hermes kanban reassign <task_id> <profile> --reclaim
```

Add comments documenting the failure and the intended retry path.

## Monitoring Guidance

Prefer event-driven monitoring over blind polling.

Good:

```bash
hermes kanban watch
hermes kanban tail <task_id>
hermes kanban runs <task_id>
hermes kanban diagnostics
```

Avoid repeated `sleep && hermes kanban list` unless you only need a simple delayed status check.

Encourage workers to leave meaningful comments or heartbeats for long tasks:

```text
backend scaffold done; implementing SQLite repository
endpoints complete; running smoke tests
frontend scaffold done; wiring API client
```

## Prompt Recipes

### Atomic project planning

```text
Use the kanban-architect skill.

Goal:
<PASTE REQUIREMENT HERE>

Before creating any Kanban cards, propose an atomic DAG.

Constraints:
- Use only existing profiles from `hermes profile list`.
- No task should be larger than 3-5 tool calls.
- Each task must have assignee, parent dependencies, scope, out-of-scope, done condition, and verification method.
- Prefer functional increments over broad modules.
- Do not bundle backend + frontend in one card.
- Add a final verification/review task.
- Show the graph first and wait for approval before creating tasks.
```

### Auto-run after approval

```text
The DAG is approved. Create the Kanban cards with the dependencies exactly as shown, then start or verify the gateway and monitor with `hermes kanban watch`.
```

### Post-mortem

```text
Use kanban-architect to analyze this board/run.
Report hurdles by layer: decomposition, profile routing, workspace/auth, provider/config, worker behavior, review/verification.
Recommend concrete skill or workflow changes.
```

## Common Pitfalls

1. **Creating module blobs.** `Implement backend` hides progress and makes retries expensive. Split by scaffold, data layer, endpoint, test, and verification.
2. **Inventing profiles.** Unknown assignees silently sit in ready or fail to spawn. Always discover profiles first.
3. **Skipping preflight.** Auth and provider errors are cheaper to catch before workers launch.
4. **Linking dependencies after creation.** Create child cards with parents from the start to avoid race conditions.
5. **Over-linking.** Parallel work should remain parallel. Only link when a card truly needs another card's output.
6. **No review gate.** Code-changing tasks often need a reviewer or verification card before root completion.
7. **No done condition.** Workers need a concrete target and verification command.
8. **Treating skills as slash commands.** In TUI, skills are loaded/referenced as procedural context; use natural language or `/skill kanban-architect` if available.

## Verification Checklist

Before dispatch:

- [ ] Board/project selected.
- [ ] `hermes profile list` checked.
- [ ] Assignees are real profiles.
- [ ] Provider/model works for each worker profile.
- [ ] GitHub/auth requirements checked if needed.
- [ ] DAG shown to user for complex projects.
- [ ] Cards are atomic enough.
- [ ] Parent links represent true dependencies.
- [ ] Every card has done condition and verification.
- [ ] Final review/verification card exists.

After execution:

- [ ] All implementation cards done.
- [ ] Verification/review cards done or explicitly waived.
- [ ] Root card completed with summary.
- [ ] Hurdles documented as comments or post-mortem notes.
