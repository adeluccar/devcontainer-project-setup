---
name: to-issues
description: Break a plan, spec, or PRD into independently-grabbable GitHub Issues using tracer-bullet vertical slices. Use when user wants to convert a plan into issues, create implementation tickets, or break down work into issues.
---

Break a plan into independently-grabbable issues using vertical slices (tracer bullets).

The project issue tracker is **GitHub Issues** on this repo.

## Process

### 1. Gather context

Work from whatever is already in the conversation. If the user passes an issue reference (number or URL), fetch it and read its full body and comments.

### 2. Explore the codebase (optional)

If you haven't already explored the codebase, do so to understand the current state. Issue titles and descriptions should use the project's domain vocabulary.

### 3. Draft vertical slices

Break the plan into **tracer bullet** issues. Each issue is a thin vertical slice that cuts through ALL integration layers end-to-end — NOT a horizontal slice of one layer.

Each slice is either:

- **HITL** (Human In The Loop) — requires a decision, client input, or design review
- **AFK** (Away From Keyboard) — can be implemented and merged without human interaction

Rules:

- Each slice delivers a narrow but COMPLETE path through every layer
- A completed slice is demoable or verifiable on its own
- Prefer many thin slices over few thick ones
- Prefer AFK over HITL where possible

### 4. Quiz the user

Present the proposed breakdown as a numbered list. For each slice, show:

- **Title**: short descriptive name
- **Type**: HITL / AFK
- **Blocked by**: which other slices must complete first
- **User stories covered**: which user stories this addresses (if source material has them)

Ask:

- Does the granularity feel right?
- Are dependency relationships correct?
- Should any slices be merged or split?
- Are the HITL/AFK classifications correct?

Iterate until approved.

### 5. Publish the issues

For each approved slice, publish a new GitHub Issue using the template below. Publish in dependency order (blockers first) so you can reference real issue numbers in "Blocked by".

## Issue Template

**Title:** [short descriptive name] ([AFK] or [HITL])

**Body:**

### Parent

Link to parent issue (if source was an existing issue; omit otherwise).

### What to build

A concise description of this vertical slice. Describe end-to-end behavior, not layer-by-layer implementation. Avoid specific file paths. Exception: prototype snippets that encode a decision precisely (state machine, type shape) — inline and note they came from a prototype.

### Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

### Blocked by

- Reference to blocking issue (or "None — can start immediately")

Do NOT close or modify any parent issue.
