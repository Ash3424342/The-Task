---
title: "Claude Code Bootstrap Prompt"
status: done
priority: high
owner: me
deliverable: "Ready-to-paste prompt that bootstraps a new Claude Code session with full project context"
sources:
  - "[[raw/claude_code_bootstrap_prompt.md]]"
  - "[[raw/claude_code_prompt.md]]"
ai-context: "[[AI-Sessions/claude-scoping-content-engine-compass-2026-04-07]]"
type: deliverable
---

# Claude Code Bootstrap Prompt

Two versions available:

## Bootstrap Prompt (full context transfer)
**File:** [[raw/claude_code_bootstrap_prompt.md]] (8.6KB)

Transfers everything from the scoping session into a new Claude Code session:
- Both tasks with full context
- Company context (Groww, sub-brands, systems)
- 4-box pipeline with ownership
- Content Engine 6 components + tech stack
- Compass Skills state (16 existing, enhancement priorities)
- Critical data references (join keys, schemas, stale tables, guardrails)
- Key corrections from scoping
- Prior work section (TODO — needs to be filled in by user)
- Ends with "explore my project directory and tell me what exists"

## Shorter Prompt (context + reconciliation)
**File:** [[raw/claude_code_prompt.md]] (4.7KB)

Same context but more concise. Lists 6 key corrections from scoping. Includes critical data references inline. Ends with "explore my project files and assess what's reusable."

## Usage
Copy-paste either file into a fresh Claude Code session opened at the project directory. Claude will audit existing code against the scoped plan and recommend next steps.
