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

## 🌟 Consuming Example: Building "TodoMaster" (TUI Perspective)

Here is how a human interacts with the Hermes TUI to build a full-stack Todo app with **React + FastAPI + SQLite** using this skill.

### 1. The Setup
The user uploads or creates a `prd.md` file in the workspace detailing the TodoMaster requirements (CRUD, categories, due-dates).

### 2. The Interaction Flow

**Phase A: Triage (Defining the Goal)**
**User Prompt:** *"I've provided `prd.md`. Please use the Kanban Architect skill to create a triage card for this project."*
**Agent Action:** The agent reads the PRD and executes `hermes kanban create --triage`.
**TUI Result:** A new card appears in the **Triage** column.

**Phase B: Decomposition (Building the Workforce)**
**User Prompt:** *"Now, decompose the root triage card into a specialized workforce DAG."*
**Agent Action:** The agent executes `hermes kanban decompose <task_id>`.
**TUI Result:** The single triage card vanishes, and a graph of 4-6 specialized tasks (DB $\rightarrow$ Backend $\rightarrow$ Frontend $\rightarrow$ QA) appears in the **Todo** column, correctly linked by dependencies.

**Phase C: Execution (Launching the Build)**
**User Prompt:** *"Launch the gateway and monitor the build progress."*
**Agent Action:** The agent executes `hermes gateway start` and `hermes kanban watch`.
**TUI Result:** You see tasks moving in real-time: `Ready` $\rightarrow$ `Running` $\rightarrow$ `Done`.

### 3. Final Result
**Agent Report:** *"The TodoMaster app is now fully implemented. All integration tests passed, and the app is accessible at localhost:5173. The root task is marked as Done."*

## 📐 Architecture
- **Paradigm:** Runtime-Structured Orchestration.
- **State:** Durable SQLite backend via `kanban_db`.
- **Isolation:** Uses Git Worktrees for parallel development.
