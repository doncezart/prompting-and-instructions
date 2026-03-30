# GitHub Copilot Instructions

A structured context framework for GitHub Copilot. Gives Copilot a consistent session
protocol, a defined set of reference files, and clear rules for how to behave — so it
works with your project's actual state rather than guessing.

---

## What it does

- Forces Copilot to read your project context before writing a single line of code
- Defines a set of reference files (`BRIEF.md`, `TASKS.md`, `DECISIONS.md`, etc.) that
  serve as the source of truth for every session
- On blank projects, interviews you like a project manager — building the full framework
  from your answers before a single line of code is written
- On existing projects with no docs, infers the context by analyzing the codebase itself
- Enforces consistent behavior: no library introductions without approval, no reopening
  settled decisions, no destructive operations without confirmation
- Closes every session by updating a changelog, an AI action log, and a feature map —
  automatically, unprompted

## Files it manages

| File | What it's for |
|---|---|
| `docs/BRIEF.md` | Stack, architecture, constraints — the core context file |
| `docs/TASKS.md` | Work queue: In Progress, Up Next, Backlog, Done |
| `docs/DECISIONS.md` | Architectural decisions with rationale, never re-argued |
| `docs/CONVENTIONS.md` | Coding standards and approved libraries |
| `docs/CHANGELOG.md` | Version history in Keep a Changelog format |
| `docs/FUNCTIONALITY.md` | Full feature map: what every feature does, how, and what it calls |
| `docs/AI_LOG.md` | Append-only log of every action Copilot takes |
| `.env.example` | Placeholder env vars — no real secrets ever committed |

---

## How to use

**1. Copy the instructions file into your repo:**

```
.github/copilot-instructions.md
```

That path is where GitHub Copilot looks for workspace instructions automatically.
No configuration needed beyond that.

**2. Start a Copilot chat with this prompt:**

```
Run the session start protocol.
```

Copilot will assess the project state and take one of three paths:

---

### Path A — Blank or skeleton project (Discovery Mode)

Copilot enters **Project Discovery Mode** and interviews you conversationally — 2
questions at a time, adapting to your answers — covering: purpose, users, core features,
stack, data, external services, constraints, and out of scope. When it has enough, it
summarizes back and asks for a quick yes/no confirmation before generating anything. On
confirmation, it scaffolds the full `docs/` structure with `BRIEF.md` already filled in
from the interview, and suggests an initial task backlog.

---

### Path B — Existing project, no docs yet

If real source files exist but `docs/BRIEF.md` is missing or blank, Copilot analyzes
the codebase itself — scanning manifests, config files, folder structure, and source
files — and fills in `BRIEF.md` from what it finds. Sections it can't confidently
determine are marked `<!-- could not infer — please review -->`. It then pauses and
asks you to review before proceeding.

---

### Path C — Existing project with docs

Normal flow. Copilot reads `BRIEF.md` and `TASKS.md` at the start of every session and
gets to work. Incomplete placeholder sections are flagged at the top of the first
response, but don't block progress.

---

**3. Work normally.**

At the end of each session where meaningful changes occurred, Copilot outputs changelog
entries, AI log lines, and functionality updates — ready to paste in, unprompted.

---

## Key behaviors to know

- **BRIEF.md is a living document.** It can be generated from an interview, inferred
  from the codebase, or written manually. It's not locked — update it freely as the
  project evolves.
- **DECISIONS.md is locked.** Copilot will not re-argue a recorded decision unless you
  explicitly ask it to revisit one. If a decision looks outdated, it flags it once and
  moves on.
- **Conflict priority:** If reference files contradict each other or the actual codebase,
  Copilot follows: `codebase > DECISIONS.md > BRIEF.md > CONVENTIONS.md > TASKS.md`
  and flags the conflict before proceeding.
- **Destructive operations always pause.** Dropping tables, deleting files, overwriting
  configs — Copilot describes what it's about to do and waits for your approval.
- **AI_LOG.md is append-only.** One line per action, never edited retroactively.
  Format: `YYYY-MM-DD | action | file(s)`
