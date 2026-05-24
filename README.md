# Hermes Kanban Architect Skill

`kanban-architect` is a Hermes skill for converting PRDs, specs, and high-level project goals into durable Hermes Kanban task graphs.

It focuses on **atomic, profile-routed, dependency-aware decomposition** instead of vague module-level cards like `Implement backend` or `Build frontend`.

## What It Does

The skill guides an agent through:

1. **Preflight** – check profiles, provider health, GitHub auth, gateway status, and board context.
2. **Triage** – create a root requirement card.
3. **Graph proposal** – propose an atomic DAG before creating cards for complex projects.
4. **Card creation** – create Kanban cards with assignees, parents, scope, done conditions, and verification commands.
5. **Execution monitoring** – use gateway dispatcher, `kanban watch`, `runs`, `log`, and diagnostics.
6. **Recovery / post-mortem** – diagnose provider, auth, profile, decomposition, or worker issues.

## Why This Version Exists

Earlier usage showed that broad decomposition works, but creates friction:

```text
Bad graph:
Spec
Project structure
Backend
Frontend
Root
```

Those cards are too coarse. Workers must both plan and implement, progress is opaque, and retries are expensive.

This skill now encourages atomic implementation graphs:

```text
Root: full-stack app
├── Write API/data spec
├── Initialize repo/project structure
├── Create backend scaffold
├── Implement database layer
├── Implement GET endpoint
├── Implement POST endpoint
├── Add backend smoke tests
├── Create frontend scaffold
├── Create API client
├── Implement list UI
├── Implement mutation UI
├── End-to-end verification
└── Final review/root completion
```

## Installation

Clone this repository:

```bash
gh repo clone rajivmehtaflex/kanban-architect
```

Copy the skill into your Hermes skills directory:

```bash
cp -r kanban-architect/skills/* ~/.hermes/skills/
```

Then start a new Hermes session or reload skills if your platform supports it:

```text
/reload-skills
```

## TUI Usage

In Hermes TUI, use natural language or explicitly load the skill:

```text
/skill kanban-architect
```

Then prompt:

```text
Use the kanban-architect skill.
Read ./prd.md and propose an atomic Kanban DAG first.
Do not create cards until I approve.
Use only profiles from `hermes profile list`.
```

After approval:

```text
Create the approved Kanban cards with dependencies, then start or verify gateway and monitor execution.
```

## Prompt Showcase: Using This Skill with Hermes Agent

You can use this skill from the Hermes Agent CLI, TUI, or any connected gateway session. The safest workflow is to first ask Hermes to **load the skill and propose the DAG**, then approve the graph before it creates Kanban cards.

### 1. Start Hermes with the skill preloaded

```bash
hermes -s kanban-architect
```

Or inside an existing Hermes TUI session:

```text
/skill kanban-architect
```

### 2. Prompt Hermes to plan, but not create cards yet

```text
Use the kanban-architect skill.

Project:
Build a full-stack Todo app with FastAPI, SQLite, React, and Vite.
Users should be able to create, list, update, complete, and delete todos.
The backend should expose a JSON API and the frontend should consume it.

Before creating any Kanban cards, do the following:
1. Run preflight checks for profiles, assignees, gateway status, provider health, and GitHub auth if needed.
2. Use only existing Hermes profiles from `hermes profile list`.
3. Propose an atomic Kanban DAG first.
4. Do not create cards until I approve the graph.

DAG requirements:
- No task should be larger than 3-5 tool calls for an oriented worker.
- Split backend work by scaffold, database layer, API endpoints, and backend tests.
- Split frontend work by scaffold, API client, UI components, and integration verification.
- Do not bundle backend and frontend work in the same card.
- Every task must include assignee, parent dependencies, scope, out-of-scope, done condition, and verification method.
- Add a final verification/review task before root completion.
```

### 3. Approve the proposed graph

After Hermes shows the graph, reply with something like:

```text
Approved. Create the Kanban cards exactly as proposed, preserving the dependencies and assignees. Then start or verify the gateway dispatcher and monitor progress with `hermes kanban watch`.
```

### 4. Prompt Hermes to analyze a failed or coarse run

```text
Use the kanban-architect skill to analyze this Kanban run.

Board/task context:
<PASTE BOARD, TASK IDS, OR LOG SUMMARY>

Report issues by layer:
- decomposition granularity
- profile routing
- workspace or GitHub auth
- provider/model config
- worker behavior
- verification/review quality

Then recommend concrete fixes and, if needed, propose a better atomic DAG before creating or modifying cards.
```

### 5. Generic reusable prompt

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

## Core Hermes Commands

```bash
hermes profile list
hermes kanban assignees
hermes doctor
gh auth status

hermes kanban create "<title>" --triage --body "$(cat prd.md)"
hermes kanban decompose <task_id>
hermes gateway start
hermes kanban watch
hermes kanban runs <task_id>
hermes kanban log <task_id>
hermes kanban diagnostics
```

## Repository Layout

```text
skills/
└── kanban-architect/
    └── SKILL.md
skills.sh.json
README.md
```
