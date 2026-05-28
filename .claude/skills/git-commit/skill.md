---
name: git-commit
description: Create standardized Conventional Commits from current changes. Detects WIP commits and squashes them. Splits large diffs into atomic commits. Use when user says /commit, wants to commit changes, or asks to commit work.
---

# Git Commit Skill

## What this skill does

1. **Blocks** commits on `main` — tells you to branch first
2. **Detects WIP** — if the last commit is a WIP commit, resets it to absorb those changes
3. **Analyzes all changes** and groups them into atomic commits by concern
4. **Generates** a Conventional Commit message per group
5. **Shows each message** for your approval before executing
6. **Commits** the approved message

## Quick start

User says `/commit` → skill runs the full workflow automatically. No other input needed unless approval is requested.

---

## Workflow (run in order)

### Step 1 — Branch check

```bash
git branch --show-current
```

If the result is `main`: **hard stop**. Tell the user to create a feature branch first (`git checkout -b feat/...`). Do not proceed.

### Step 2 — WIP detection

```bash
git log -1 --format="%s"
```

If the subject matches `/^wip[\s:!(\[]?/i` (case-insensitive, any form: `wip`, `WIP:`, `wip!`, `[WIP]`, `wip(nav)`, etc.):

- Show the user: "Last commit is a WIP commit — absorbing it into the new commit(s)."
- Run: `git reset HEAD~1` (keeps all changes in working tree)

### Step 3 — Inspect changes

```bash
git status --short
git diff HEAD
```

Read both outputs to understand:

- Which files changed
- What the changes actually do (read the diff content, not just filenames)
- Whether changes span multiple independent concerns

### Step 4 — Plan atomic commits

Group the changed files into **logical units of work**. Each group should answer "why did these files change together?"

**Rules for splitting:**

- Changes to `src/components/Nav.astro` + `src/pages/index.astro` (nav-related) → one commit
- Changes to `src/styles/fonts.css` + `Layout.astro` (font-related) + `package.json` (unrelated dep bump) → two commits
- When in doubt, err toward **fewer, broader commits** over many tiny ones
- Config/tooling changes (`astro.config.mjs`, `.prettierrc`, devcontainer) always get their own commit, separate from feature work

For each group, note:

- Files to stage
- The commit type, scope, and subject you'll propose

### Step 5 — For each commit group (in sequence)

#### 5a. Generate the message

Format: `type(scope): subject`

**Types:**

- `feat` — new feature or visible change
- `fix` — bug fix
- `chore` — maintenance, config, deps (no production code change)
- `docs` — documentation only
- `style` — formatting, whitespace (no logic change)
- `refactor` — code restructure, no behavior change
- `test` — adding or fixing tests
- `perf` — performance improvement
- `ci` — CI/CD config
- `build` — build system, tooling

**Common scopes** (not exhaustive — coin new ones when genuinely new territory):
`hero` · `nav` · `footer` · `contact` · `gallery` · `legal` · `layout` · `fonts` · `images` · `deps` · `ci` · `config`

Scope is optional — omit it when the change is truly cross-cutting.

**Subject rules (all required):**

- Lowercase after the colon
- Max 72 characters total (including `type(scope): `)
- No trailing period
- Imperative mood: "add", "fix", "remove" — not "added", "fixes", "removing"

**Body** (optional — include when subject alone isn't self-explanatory):

- Blank line after subject
- Explain _why_, not _what_ (the diff shows what)
- Wrap at 72 characters

**Footer:**

- `Closes #N` — include if this commit closes a GitHub issue (mandatory when applicable)
- `BREAKING CHANGE: description` — if the commit introduces a breaking change

**Never include:**

- `Co-Authored-By:` lines of any kind

#### 5b. Lint the message

Before showing it to the user, verify:

- [ ] Subject matches `type(scope?): subject` pattern
- [ ] Type is one of the 10 allowed types
- [ ] Subject is lowercase after the colon
- [ ] Subject ≤ 72 characters
- [ ] Subject does not end with a period
- [ ] Subject uses imperative mood (check for common non-imperative words: adds, added, adding, fixes, fixed, fixing, removes, removed, removing)
- [ ] Body lines ≤ 72 characters (if body present)
- [ ] `Closes #N` present if closing a known issue

Fix any violations before showing the message.

#### 5c. Show for approval

Display the full message in a code block and ask: "Commit with this message?"

Wait for explicit confirmation before proceeding. If the user edits or rejects, revise and re-lint.

#### 5d. Stage and commit

```bash
git add <files in this group>
git commit -m "type(scope): subject" -m "body paragraph" -m "Closes #N"
```

Use separate `-m` flags for subject, body, and footer so git formats the message correctly. If there's no body, skip that `-m`.

Then move to the next group (Step 5a).

---

## Edge cases

**Nothing to commit:** `git status` shows clean tree → tell the user, exit.

**Already fully staged (`git add -A` was run manually):** Proceed from Step 4 using what's staged.

**Ambiguous issue number:** If the user mentions an issue but hasn't given a number, ask before committing.

**Merge conflicts present:** Stop and tell the user to resolve conflicts first.

---

## Example output

```
feat(hero): add Cloudinary image as video placeholder

Use a static Cloudinary image in the hero background until
the client delivers the real video asset. The <a href="#video">
anchor is preserved for later wiring.

Closes #18
```

```
chore(config): add commitlint config for Conventional Commits
```
