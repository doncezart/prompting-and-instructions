# Copilot instructions

This project uses a structured context framework. The files described below are the
source of truth for all project work. Always prioritize them over assumptions.

---

## Session start protocol

At the start of every session, before writing any code or making any suggestions:

1. Check that `docs/BRIEF.md` exists and has content beyond its placeholder headers.
   - If the file does not exist at all, stop and notify the user: "BRIEF.md is missing
     — ask me to scaffold the project first."
   - If it exists but is empty or contains only placeholder text (e.g. "TODO", "TBD",
     `<!-- fill this in -->`), do not stop. Instead, analyze the project yourself:
     scan the codebase, `package.json` / `requirements.txt` / equivalent manifest,
     folder structure, existing source files, and any config files to infer the stack,
     architecture, and purpose. Fill in `docs/BRIEF.md` based on what you find, mark
     any section you could not confidently determine with
     `<!-- could not infer — please review -->`, then notify the user: "BRIEF.md was
     empty — I've filled it in from the codebase. Please review it before we continue,
     especially any sections marked for review." Do not proceed with the user's request
     until they confirm or correct the file.
   - If it exists and has real content but some sections are still placeholder text,
     proceed but list the incomplete sections at the top of your first response.
2. Read `docs/BRIEF.md` in full.
3. Read `docs/TASKS.md` to identify the current priority.
4. Proceed based on the user's instruction, grounded in those two files.

---

## Reference files

| File | Purpose | When to read |
|---|---|---|
| `docs/BRIEF.md` | Tech stack, architecture, constraints, scope | Every session, always |
| `docs/TASKS.md` | Current priorities and backlog | Every session, always |
| `docs/DECISIONS.md` | Past architectural decisions | Before any structural change |
| `docs/CONVENTIONS.md` | Coding standards, approved libraries | Before writing any code |
| `docs/CHANGELOG.md` | History of meaningful changes | When unsure if something is already done |
| `docs/FUNCTIONALITY.md` | Full feature map by category/page | Before adding or modifying any feature |
| `docs/AI_LOG.md` | Concise log of every AI action | Append to at end of every session |

---

## Behavior rules

- **Never assume the stack.** If something is not in `BRIEF.md`, ask before proceeding.
- **Never introduce libraries or tools** not listed in `CONVENTIONS.md` without flagging
  it explicitly and getting approval first.
- **Never re-open a decision** that is already recorded in `DECISIONS.md` unless the
  user explicitly asks to revisit it. However, if a recorded decision appears outdated,
  broken, or contradicted by the actual codebase, flag it once without re-arguing the case.
- **Prefer implementation over explanation.** Deliver working code and concrete next
  steps. Explain only when asked.
- **Flag stale context.** If `BRIEF.md` describes a stack or architecture that appears
  inconsistent with the actual codebase, point it out before proceeding.
- **Conflict resolution order.** If any two reference files contradict each other, or
  if a reference file contradicts the actual codebase, use this priority to decide
  which to trust and flag the conflict before proceeding:
  `Actual codebase > DECISIONS.md > BRIEF.md > CONVENTIONS.md > TASKS.md`
- **Destructive operations require confirmation.** The `prefer implementation` rule
  does not apply to irreversible actions (deleting data, dropping schema, removing
  files, overwriting configs). For these, describe exactly what will happen and wait
  for explicit approval before executing.

---

## Session end protocol

After any session in which one or more of the following occurred, always perform all
steps below unprompted:

- A new file was created
- An existing file was structurally changed
- A new dependency was introduced
- A decision was made that is not yet recorded in `DECISIONS.md`

**Steps:**

1. Provide a `docs/CHANGELOG.md` entry in the format specified below, ready to paste in.
2. Append new lines to `docs/AI_LOG.md` covering every action taken this session,
   in the format specified below.
3. Update `docs/FUNCTIONALITY.md` for every feature added or changed this session,
   in the format specified below.
4. Suggest any updates to `docs/TASKS.md` (move completed items, flag blockers, surface
   anything that emerged during the session that should be tracked).
5. Flag if any changes made during the session should be reflected in `docs/BRIEF.md`
   or `docs/DECISIONS.md`, and summarize what should be updated.

---

## CHANGELOG format

Use this format for all `docs/CHANGELOG.md` entries:

```
## [Unreleased] or ## [x.y.z] — YYYY-MM-DD

### Added
- Past-tense, verb-first description ("Added Facebook OAuth to /auth/facebook")

### Changed
- Past-tense, verb-first description ("Refactored DB connection pool in src/db.ts")

### Fixed
- Past-tense, verb-first description ("Fixed null check on user.email in auth middleware")

### Removed
- Past-tense, verb-first description ("Removed deprecated /api/v1/login route")
```

Rules:
- Use `[Unreleased]` when no version is being cut; use a semver tag when releasing.
- Use today's date in `YYYY-MM-DD` format.
- Only include subsections that have entries — omit empty ones.
- One line per item. Keep descriptions specific enough to be useful without reading the diff.

---

## AI_LOG format

`docs/AI_LOG.md` is an append-only file. Add one line per action at the end of every
session. Never edit or delete previous lines.

```
YYYY-MM-DD | action description | file(s)
```

Rules:
- Keep each line under 120 characters total.
- Action description: verb-first, past tense, as short as possible.
- List only the files directly affected. Omit files that were only read.
- If multiple files are affected by the same action, separate them with `, `.

Examples:
```
2025-06-01 | added db connection | src/db.ts
2025-06-01 | created Facebook OAuth route | src/routes/auth.ts, src/controllers/auth.ts
2025-06-01 | added User model | src/models/user.ts
2025-06-02 | refactored token validation into middleware | src/middleware/auth.ts
2025-06-02 | dropped legacy /api/v1/login route | src/routes/api.ts
```

---

## FUNCTIONALITY format

`docs/FUNCTIONALITY.md` is the authoritative map of everything the project does.
Organize it by page, domain, or feature category — whichever is most natural for
the project. Update it whenever a feature is added or meaningfully changed.

Each feature entry follows this structure:

```markdown
## [Category or Page Name]

### Feature Name
- **What it does:** One or two sentences describing the user-facing behavior.
- **Entry point:** `path/to/file.ext` (the file where this feature begins executing)
- **Key functions / components:** `functionName()`, `ComponentName` — list the most
  important ones only, not every helper.
- **API calls:** Method, endpoint, brief payload/response shape.
  Example: `POST /api/auth/facebook` → `{ code }` → `{ token, user }`
- **Data flow:** Brief description of how data moves through the system for this feature.
- **Dependencies:** External services, libraries, or other features this depends on.
```

Omit any field that does not apply. For simple features, two or three fields are enough.
Do not duplicate information already in `CONVENTIONS.md` or `DECISIONS.md`.

---

## Scaffolding the project framework

When the user's intent is to initialize or create the project file structure — regardless
of exact wording — generate the following files.

**Before generating:**
- Check whether any of these files already exist with content beyond placeholder headers.
  For any file that already has real content, skip it and note it in the summary.
- If any project details are already known from the conversation (name, stack, purpose),
  use them to pre-fill the relevant sections. Do not invent anything not yet stated.

**All files go inside `docs/`**, except `README.md` which stays at the project root
(GitHub renders it there by convention).

**Files to generate:**

`README.md` *(project root)* — Public overview. Sections: project name and one-line
description, what it does, who it's for, tech stack summary, getting started steps.

`docs/CHANGELOG.md` — Version history log. Start with an `[Unreleased]` block and
empty Added / Changed / Fixed / Removed subsections. Follow the CHANGELOG format
defined in this file.

`docs/TASKS.md` — Work queue. Four sections: In Progress, Up Next, Backlog, Done.
Each with a single empty bullet. Use this task format:
`- [ ] TASK-ID: Description (optional: blocked by TASK-ID)`

`docs/BRIEF.md` — AI context file. Sections: what this project does, who it's for,
tech stack and architecture, key data models, external APIs and services, constraints
and non-negotiables, out of scope.

`docs/DECISIONS.md` — Decisions log. Include one empty template entry with fields:
Decision, Why, Alternatives considered, Date.

`docs/CONVENTIONS.md` — Coding standards. Sections: language and runtime, formatting
and style, naming conventions, folder structure, approved libraries, off-limits.

`docs/FUNCTIONALITY.md` — Feature map. Start with a single placeholder category and
one placeholder feature entry using the FUNCTIONALITY format defined in this file.

`docs/AI_LOG.md` — AI action log. Start with a single header line and one example
entry (commented out) showing the correct format.

`.env.example` *(project root)* — Placeholder environment variables with no real values.
Example: `DATABASE_URL=your_database_url_here`. Never write real secrets into this file
or any committed file.

**After generating:**
- Create the `docs/` directory if it does not exist.
- Check `README.md` for a line referencing the AI guidance framework. If one is not
  already present, append the following line to the end of the file:
  `> AI guidance framework: https://github.com/doncezart/prompting-and-instructions/tree/main/github-copilot-instructions`
- Print a summary: list every file created, every file skipped and why.
- Then prompt: "BRIEF.md has been created with placeholder sections — fill it in before
  the next session so I have full context to work with."
