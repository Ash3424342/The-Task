# How-To Guide: Your AI Manager Second Brain

## What Is This?

This vault is an **autonomous AI Manager** that lives inside Obsidian. It acts as your personal executive assistant that:

- **Ingests** everything — boss messages, Claude/Gemini session exports, deliverables, specs, notes
- **Extracts tasks** automatically and tracks them with status, priority, owner, and due dates
- **Synthesizes context** across multiple AI sessions and sources into one coherent plan
- **Generates commands** — ready-to-copy prompts you can paste into any coding agent (Cursor, Gemini, etc.)
- **Verifies deliverables** — checks completed work against boss requirements
- **Prunes its own memory** — consolidates old logs so the system stays fast and focused
- **Never loses context** — everything is linked, searchable, and compounding

Think of it as a project manager that never sleeps, never forgets, and keeps every thread connected.

---

## Vault Structure

```
The Task/
├── raw/                  ← DROP EVERYTHING HERE (immutable, never edited)
├── Tasks/                ← Auto-created task notes (one per task)
├── Projects/             ← One folder per major project
├── AI-Sessions/          ← Claude/Gemini transcripts and analyses
├── Deliverables/         ← Boss specs and verified outputs
├── Manager/              ← Your command center
│   ├── Manager-Dashboard.md   ← Live Bases views (table + kanban)
│   ├── Execution-Log.md       ← History of all Manager actions
│   └── Today-Commands.md      ← Daily prompts for coding agents
├── _Archive/             ← Auto-pruned old content
├── wiki/                 ← Concept/entity pages (Karpathy-style)
├── index.md              ← Master index with wikilinks
├── log.md                ← Global activity log
├── CLAUDE.md             ← System manifest (the brain's rules)
└── SETUP-NOTES.md        ← One-time setup checklist
```

### Key Rule: `raw/` is Sacred
- **Never edit** files in `raw/`. They are your source of truth.
- Everything else in the vault is maintained by the AI Manager and can be regenerated from `raw/`.

---

## How to Use It

### 1. Capture: Drop Files into `raw/`

Anything you want the Manager to know about goes into `raw/` as a `.md` file:

| What | How to capture |
|---|---|
| Boss message / meeting notes | Type or paste into a new `.md` file in `raw/` |
| Claude session | Export chat → save as `.md` in `raw/` |
| Gemini session | Export chat → save as `.md` in `raw/` |
| Voice note | Transcribe → save as `.md` in `raw/` |
| Screenshot | Drop image into `raw/` (or `raw/assets/`) |
| PDF / document | Convert to `.md` or drop as-is into `raw/` |
| Deliverable spec | Save as `.md` in `raw/` |

**Naming convention (recommended):** `boss-meeting-2026-04-07.md`, `claude-session-auth-flow.md`, `deliverable-api-spec.md`

### 2. Trigger: Run the Manager

Open Claude Code in this vault folder and type any of these:

```
Run Manager Skill
```
Full sweep — scans everything, updates all tasks and logs.

```
Manager: ingest raw/boss-meeting-2026-04-07.md
```
Ingest a specific file and extract tasks from it.

```
Manager review
```
Deep review — runs AutoDream (prunes old logs, merges notes, updates CLAUDE.md).

```
Manager: what's the status of the API project?
```
Query — searches the vault and gives you a synthesized answer.

### 3. Receive: Read the Output

After every run, the Manager will output:

1. **Summary** — What was ingested and what changed
2. **Task statuses** — Updated table with Task IDs, priorities, owners
3. **Execution plan** — Step-by-step: what to do next, in what order
4. **Agent prompts** — Copy-paste commands for Cursor, Gemini, or other AIs
5. **Risks/blockers** — Anything that needs your attention
6. **Sync confirmation** — Bases, Graph View, and Dispatch are up to date

### 4. Delegate: Send Commands to Other Agents

The Manager generates ready-to-copy prompts in `Manager/Today-Commands.md`. Example:

```
## Task: Implement auth middleware (Task ID: 7)
### For Cursor:
"Implement JWT auth middleware in src/middleware/auth.ts. 
Requirements: validate Bearer token, extract user ID, attach to req.user.
See spec in Deliverables/auth-spec.md."

### For Gemini:
"Research best practices for JWT refresh token rotation in 2026. 
Summarize in 500 words with code examples."
```

Copy the prompt → paste into the target agent → when done, save the output back into `raw/` → re-trigger the Manager.

### 5. Review: Open Obsidian

- **Manager Dashboard** (`Manager/Manager-Dashboard.md`) — Live table and kanban views of tasks and deliverables using Obsidian Bases
- **Graph View** (sidebar icon) — Visual map of all connected notes
- **index.md** — Master catalog of everything in the vault

---

## Key Files Explained

| File | Purpose |
|---|---|
| `CLAUDE.md` | The brain's rulebook. Defines how the Manager behaves, what properties to use, and how to handle wiki pages. Read automatically on every session. |
| `.claude/skills/managing-work-context/SKILL.md` | The Manager Skill definition. Contains the detailed instructions, allowed tools, and output format. |
| `Manager/Manager-Dashboard.md` | Your daily command screen. Open in Obsidian to see Bases views (requires Bases core plugin enabled). |
| `Manager/Execution-Log.md` | Chronological history of every Manager action. Auto-pruned by AutoDream. |
| `Manager/Today-Commands.md` | Generated prompts for coding agents. Updated every Manager run. |
| `index.md` | Master index of all vault content with wikilinks. |
| `SETUP-NOTES.md` | One-time checklist for Obsidian plugin configuration. |

---

## Advanced Features

### AutoDream (Memory Pruning)
Every 10th Manager call (or on `Manager review`), the system automatically:
- Prunes stale execution logs
- Merges fragmented notes into coherent summaries
- Synthesizes learnings back into `CLAUDE.md`
- Archives old content to `_Archive/`

This prevents the vault from getting bloated and keeps the AI focused on what matters.

### Wiki / Knowledge Pages
The `wiki/` folder holds Karpathy-style entity and concept pages:
- **Entity pages:** People, tools, frameworks, companies mentioned across your sources
- **Concept pages:** Technical ideas, patterns, methodologies

The Manager creates and updates these automatically when ingesting from `raw/`. On `Manager review`, it checks for orphan pages, contradictions, and missing backlinks.

### Multi-Agent Dispatch
The Manager can create tasks with dependencies, so multiple Claude Code sessions can work in parallel:
- Task A blocks Task B → agents automatically respect the order
- Each task has clear success metrics from the boss requirements

### Verification Loop
When a task is marked "Ready for Review", the Manager audits the deliverable against the original boss requirements in `raw/`. Only marks "Completed" after verification passes.

### Obsidian MCP (Semantic Search)
With the Local REST API plugin enabled, Claude Code can query your vault using natural language instead of scanning files manually. This makes the Manager faster and more accurate on large vaults.

---

## Daily Workflow (1-2 Minutes)

```
Morning:
1. Drop any new files into raw/
2. Open Claude Code → type "Run Manager Skill"
3. Read the output → check Today-Commands.md
4. Copy prompts into your coding agents

During the day:
5. Save agent outputs back into raw/
6. Re-trigger Manager as needed

Weekly:
7. Type "Manager review" for AutoDream cleanup
```

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Bases views are empty in Dashboard | Enable the **Bases** core plugin in Obsidian Settings → Core plugins |
| Manager doesn't find new files | Make sure files are saved as `.md` in `raw/` (not subfolders) |
| MCP server not connecting | Ensure Obsidian is open with Local REST API enabled, then restart Claude Code |
| Skill not triggering | Make sure you're running Claude Code from the vault folder (`cd` to it first) |
| Graph View shows no connections | Run the Manager at least once — it creates wikilinks between notes |

---

## What Properties Mean

Every task/project note created by the Manager has this YAML frontmatter:

```yaml
status: todo          # todo → in-progress → blocked → review → done
priority: high        # high, medium, or low
owner: Claude         # who's responsible (me, Claude, Gemini, Cursor, etc.)
due: 2026-04-15       # deadline
deliverable: "..."    # exact boss requirement this fulfills
sources: [[raw/...]]  # link to the original raw input
ai-context: [[...]]   # link to relevant AI session analysis
task-id: "7"          # native Dispatch Task ID
execution-log: "..."  # brief history of what happened
```

These properties power the Bases views in the Dashboard — filtering, sorting, and grouping happen automatically.
