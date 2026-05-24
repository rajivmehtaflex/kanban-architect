---
name: "Hermes Kanban Architect"
description: "Transforms PRDs/Specs into autonomous task graphs via Triage and Decomposition."
---

## When to use this skill
Use this skill when you are provided with a `prd.md`, `spec.md`, or a high-level requirements document and need to initialize an autonomous build pipeline. This skill is specifically for **Level 3: Autonomous Orchestration**, moving away from manual task lists toward a "what-is-needed" goal-oriented approach.

## Operational Workflow

### Phase 1: Requirement Ingestion (Triage)
1. **Parse Input:** Read the provided `prd.md` or `spec.md`. 
2. **Extract Goal:** Generate a concise, imperative title (max 80 chars) and extract the core requirements for the body.
3. **Create Triage Card:** Use the CLI to drop the requirement into the Triage column.
   ```bash
   hermes kanban create "[Title]" --triage --body "[Full requirements from PRD/Spec]"
   ```
4. **Identify Task ID:** Capture the resulting task ID (e.g., `t_root_001`).

### Phase 2: Autonomous Decomposition
1. **Trigger Decomposer:** Instead of manually creating child tasks, invoke the decomposition engine.
   ```bash
   hermes kanban decompose <task_id>
   ```
2. **Verify DAG Structure:** Ensure the decomposer has fanned out the root task into a Directed Acyclic Graph (DAG).
   - Run `hermes kanban list` to see child tasks.
   - Run `hermes kanban show <task_id>` to verify the parent-child links.
3. **Validate Assignees:** Check that the decomposer routed tasks to the correct specialist profiles (e.g., `db-expert`, `backend-dev`, `frontend-dev`) based on their descriptions.

### Phase 3: Orchestration & Execution
1. **Configure Orchestrator:** Ensure the orchestrator profile is set to a profile capable of judging completion.
   ```bash
   hermes config set kanban.orchestrator_profile default
   ```
2. **Launch Dispatcher:** Start the gateway to allow the embedded dispatcher to claim `ready` tasks.
   ```bash
   hermes gateway start
   ```
3. **Monitor Pipeline:** Stream board events to track the flow from `todo` $\rightarrow$ `ready` $\rightarrow$ `running` $\rightarrow$ `done`.
   ```bash
   hermes kanban watch
   ```

## Core Code Patterns
- **Triage-to-Todo:** Always use `decompose` over `specify` for complex PRDs to ensure specialization and parallelism.
- **Dependency Logic:** Remember that child tasks are only promoted to `ready` when **all** their parents are `done`.
- **The Root Loop:** The root task only becomes `ready` after all children are `done`. The orchestrator then performs the final quality gate check.

## Constraints & Gotchas
- **Profile Descriptions:** The decomposer routes based on profile **descriptions**, not just names. If routing is incorrect, use `hermes profile edit <name>` to tighten the description.
- **Cycle Detection:** If `decompose` fails, it is likely due to a circular dependency in the LLM's proposed graph; the operation will abort atomically.
- **Workspace Isolation:** Ensure coding tasks are created with `--workspace worktree` (this is usually handled by the decomposer's default logic, but verify via `hermes kanban show`).
- **Protocol Violations:** If a worker exits without calling `kanban_complete`, the dispatcher will auto-block the task.
