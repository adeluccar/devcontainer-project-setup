---
name: to-prd
description: Turn the current conversation context into a PRD and publish it to the project issue tracker. Use when user wants to create a PRD from the current context.
---

Turn the current conversation context and codebase understanding into a PRD. Do NOT interview the user — just synthesize what you already know.

The project issue tracker is **GitHub Issues** on this repo.

## Process

1. Explore the repo to understand the current state of the codebase, if you haven't already. Use the project's domain vocabulary throughout the PRD.

2. Sketch out the major modules you will need to build or modify to complete the implementation. Actively look for opportunities to extract deep modules that can be tested in isolation.

   A deep module encapsulates a lot of functionality in a simple, testable interface which rarely changes.

   Check with the user that these modules match their expectations. Check which modules they want tests written for.

3. Write the PRD using the template below, then publish it as a GitHub Issue with the label `ready-for-agent`.

## PRD Template

### Problem Statement

The problem the user is facing, from the user's perspective.

### Solution

The solution, from the user's perspective.

### User Stories

A numbered list of user stories in the format:

1. As a [role], I want [feature], so that [benefit].

Cover all aspects of the feature extensively.

### Implementation Decisions

List of decisions made, including:

- Modules to build/modify
- Interfaces to modify
- Architectural decisions
- Schema changes
- API contracts

Do NOT include specific file paths. Exception: if a prototype produced a snippet that encodes a decision more precisely than prose (state machine, type shape), inline it and note it came from a prototype.

### Testing Decisions

- What makes a good test (only test external behavior, not implementation details)
- Which modules will be tested
- Prior art for the tests in the codebase

### Out of Scope

What is explicitly not covered by this PRD.

### Further Notes

Any additional context.
