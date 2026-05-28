---
name: handoff
description: Compact the current conversation into a handoff document so a fresh agent can continue the work. Use when user wants to hand off, wrap up a session, or prepare context for the next session.
argument-hint: What will the next session be used for?
---

Write a handoff document summarising the current conversation so a fresh agent can continue the work.

Save to the OS temporary directory (`/tmp/`) — not the current workspace.

## Rules

- **Don't duplicate** content already captured in other artifacts (PRDs, plans, ADRs, issues, commits, diffs) — reference them by path or URL instead
- **Redact** any sensitive information (API keys, passwords, PII)
- If the user passed arguments, treat them as a description of what the next session will focus on and tailor the doc accordingly

## Document structure

1. **Current state** — what was accomplished this session, linked to relevant issues/commits/PRs where possible
2. **In-progress work** — anything started but not finished, with exact file paths and what remains
3. **Next steps** — what the next agent should do first, in priority order
4. **Suggested skills** — which skills the next agent should invoke (e.g. `/to-prd`, `/grill-me`, `/to-issues`)
5. **Key context** — any decisions made this session that aren't captured elsewhere (architectural choices, client constraints, blockers)
