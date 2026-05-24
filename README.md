# Hermes Kanban Architect Skill

This skill implements **Level 3: Autonomous Orchestration** for the Hermes Kanban system. It allows an AI agent to transform high-level product requirements (PRDs or Specs) into a durable, autonomous agentic workforce.

## 🚀 Capabilities
Instead of manually creating task lists, this skill enables the agent to:
- **Ingest Requirements:** Parse `prd.md` or `spec.md` files.
- **Triage:** Automatically create "Triage" cards for raw requirements.
- **Decompose:** Use the Hermes Decomposition Engine to fan out a single goal into a Directed Acyclic Graph (DAG) of specialized tasks.
- **Orchestrate:** Route tasks to the best-fit profiles (e.g., `db-expert`, `backend-dev`, `frontend-dev`) and manage the execution lifecycle.

## 🛠 Installation

### Manual Installation
To add this skill to your Hermes Agent instance:
1. Clone this repository:
   ```bash
   git clone https://github.com/rajivmehtaflex/kanban-architect.git
   ```
2. Copy the `skills/` directory into your agent's skills folder:
   ```bash
   cp -r kanban-architect/skills/* ~/.hermes/skills/
   ```

### skills.sh Standard
This repository is fully compatible with the `skills.sh` open standard. You can reference this repository in any agent environment that supports `skills.sh` curation metadata.

## 📖 How to Use
Once installed, you can simply provide the agent with a requirements file:
**User:** *"Study prd.md and use the Kanban Architect skill to start the build."*

**Agent Workflow:**
1. **Triage:** `hermes kanban create "..." --triage --body "..."`
2. **Decompose:** `hermes kanban decompose <task_id>`
3. **Execute:** `hermes gateway start` $\rightarrow$ `hermes kanban watch`

## 📐 Architecture
- **Paradigm:** Runtime-Structured Orchestration.
- **State:** Durable SQLite backend via `kanban_db`.
- **Isolation:** Uses Git Worktrees for parallel development.

## 🌟 Consuming Example: Building "TodoMaster"

Here is a real-life step-by-step example of how an agent uses this skill to build a full-stack Todo app with **React + FastAPI + SQLite**.

### 1. The Input
The user provides a `prd.md` file:
> "Build TodoMaster: A Todo app. Tech stack: React (Frontend), FastAPI (Backend), SQLite (DB). Features: CRUD todos, categories with colors, and due-date tracking."

### 2. Step-by-Step Agent Execution

**Step A: Triage (The Entry Point)**
The agent extracts the goal and creates a triage card:
```bash
hermes kanban create "Build TodoMaster Full-stack App" \n  --triage \n  --body "Build a Todo app with React, FastAPI, and SQLite. Must include CRUD, color-coded categories, and due-date tracking."
```
*Result: Task `t_root_001` is created in the Triage column.*

**Step B: Autonomous Decomposition (The Magic)**
The agent invokes the decomposition engine to create the specialized workforce:
```bash
hermes kanban decompose t_root_001
```
*The agent (as the Architect) generates a DAG:*
- `t_002`: **DB Expert** $ightarrow$ Create SQLite schema & tables.
- `t_003`: **Backend Dev** $ightarrow$ Implement FastAPI endpoints (Depends on `t_002`).
- `t_004`: **Frontend Dev** $ightarrow$ Build React UI & API integration (Depends on `t_003`).
- `t_005`: **QA Agent** $ightarrow$ Write integration tests (Depends on `t_003` & `t_004`).

**Step C: Orchestrated Execution**
The agent starts the dispatcher and monitors the pipeline:
```bash
hermes gateway start
hermes kanban watch
```
*Execution Flow:*
`DB Expert` finishes $ightarrow$ `Backend Dev` starts $ightarrow$ `Frontend Dev` starts $ightarrow$ `QA Agent` validates $ightarrow$ `Orchestrator` performs final review.

### 3. Final Result
The agent reports: 
*"The TodoMaster app is now fully implemented. All integration tests passed, and the app is accessible at localhost:5173. The root task `t_root_001` is marked as Done."*