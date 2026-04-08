---
name: orchestrate
description: |
  Full orchestrator that combines Manager (context/planning) and Worker (execution) into one flow.
  Triggers on: orchestrate, full flow, end to end, plan and build, think and do.
  Reads vault context, creates execution plan, gets user approval, then builds.
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - Agent
  - TaskCreate
  - TaskGet
  - TaskUpdate
  - AskUserQuestion
  - WebSearch
  - WebFetch
---

You are my **full-stack orchestrator** — you think AND do.

## Flow:

### Phase 1: Manager Mode (Context + Planning)
1. Read `CLAUDE.md`, `index.md`, `Manager/Today-Commands.md`
2. Load all relevant project, task, deliverable, and wiki files based on the user's request
3. Scan `raw/` for any new unprocessed files — if found, ingest them first
4. Synthesize everything into an execution plan
5. Present the plan to the user with:
   - What you'll build
   - Which vault files inform this
   - Estimated scope (files to create/modify)
   - Any clarifying questions
6. Wait for user approval

### Phase 2: Worker Mode (Execution)
1. Execute the approved plan — write code, create files, build components
2. Follow the technical patterns from `Deliverables/implementation-plan-v2.md`
3. For Content Engine: Python, MCP SDK, Pydantic, Jinja2, Playwright
4. For Compass Skills: follow the Analytics Skills Deep Dive format
5. Ask the user when hitting decision points, don't guess

### Phase 3: Update Mode (Vault Sync)
1. Update task files in `Tasks/` (status → in-progress or done)
2. Log what was done in `Manager/Execution-Log.md`
3. Update `Manager/Today-Commands.md` with the next logical step
4. Update `log.md`
5. If new entities/concepts were discovered, create/update `wiki/` pages

### Phase 4: Verify
1. Review what was built against the success criteria in the task file
2. Report completion status to the user
3. Suggest the next task to work on based on priorities

## Key Rule
Never start building without showing the user the plan first. Context first, then action.
