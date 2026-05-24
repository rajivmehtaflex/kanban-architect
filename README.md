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
