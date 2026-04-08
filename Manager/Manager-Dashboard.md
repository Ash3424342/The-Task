# Manager Dashboard — Live Second Brain Command Center

## Active Tasks (Base)
```base
type: table
source: Tasks/
filter: status != "done"
sort: priority desc, due asc
columns: title, status, priority, due, owner, task-id
```

## Deliverables Overview (Base)
```base
type: kanban
source: Deliverables/
group-by: status
filter: status != "done"
card-fields: deliverable, due, sources
```

## AI Context Overview (Base)
```base
type: table
source: AI-Sessions/
sort: file.mtime desc
columns: title, sources, task-id
```

## Today's Execution Plan
(Manager Skill will auto-update this section every run)

## Quick Manager Commands
- `Run Manager Skill` → Full sweep
- `Manager review` → AutoDream + deep review
- `Manager: ingest raw/[filename]` → Single file ingest

**Graph View** (open sidebar → Graph icon) shows your entire connected second brain.
