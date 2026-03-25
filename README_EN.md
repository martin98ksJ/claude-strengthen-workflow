# Claude Strengthen Workflow

> Language / 语言：[中文](./README.md)｜**English**

A Claude Code workflow enhancement for full-stack development, covering the complete lifecycle from design → coding → review → debugging.

3 Subagents + 19 Skills + CLAUDE.md rules — so Claude writes conformant code from the start, not just catching issues in review.

---

## What's Included

### 3 Subagents

| Agent | Role | Trigger |
|-------|------|---------|
| `designer` | Tech spec + UI/UX design docs | New feature / large change involving 3+ files |
| `reviewer` | Review code and fix issues directly | After coding, or on explicit request |
| `debugger` | Locate root cause and apply minimal fix | Build failure, test error, unexpected behavior |

**Core design**: Fix problems directly, not just report them.

---

### 19 Skills (by tech stack)

**Frontend**
| Skill | Coverage |
|-------|---------|
| `vue-conventions` | Vue3 Composition API, Element Plus, common pitfalls |
| `react-conventions` | Hooks, component patterns, state management |
| `frontend-conventions` | Component design, four-state handling, i18n, style isolation |
| `ui-ue-guidelines` | Layout, forms, dialogs, user feedback, accessibility |
| `frontend-ui-design` | ASCII wireframes + component specs + interaction flows + i18n key inventory before coding |
| `mobile-cross-platform` | Flutter/RN/MiniApp architecture, navigation, platform adaptation |

**Backend**
| Skill | Coverage |
|-------|---------|
| `go-conventions` | Naming, error handling, concurrency, common pitfalls |
| `java-conventions` | Spring, exceptions, transactions, naming |
| `python-conventions` | Type hints, async, project structure |
| `rust-conventions` | Ownership, error handling, concurrency |
| `backend-conventions` | Layered architecture, time handling, DTO, logging & security |

**General**
| Skill | Coverage |
|-------|---------|
| `code-review` | Quality / security / performance checklist |
| `testing-strategy` | Unit/integration/E2E layers, coverage, mocking |
| `performance-checklist` | Slow queries, N+1, bundle size, caching |
| `db-api-design` | Table design, RESTful API conventions |
| `error-handling` | Unified error codes, frontend-backend error chain |
| `design-first` | Write design doc before coding workflow |
| `docker-deploy` | Dockerfile, image build, health checks |
| `env-strategy` | dev/test/pre/prod config layering |

---

### 3 CLAUDE.md Rules Added

**1. Auto-load Coding Conventions**

Automatically loads the matching skill before writing code, by file type. No duplicate loads per session:
- `.go` → go-conventions + backend-conventions
- `.vue` → vue-conventions + frontend-conventions + ui-ue-guidelines
- `.tsx/.jsx` → react-conventions + frontend-conventions + ui-ue-guidelines
- `.java / .py / .rs` → language conventions + backend-conventions
- Table design / API / tests / Docker → append relevant skill as needed

**2. Change Impact Scope**

Automatically evaluates cross-layer impact before every change:
- Changing a backend API → check if frontend callers need to update
- Changing a DB field → check ORM mapping, API response, frontend binding
- Changing a shared module → grep all callers before touching

**3. Parallel Task Execution**

Automatically identifies parallel opportunities across multiple tasks:
- Lists files involved in each task
- No overlap → launches multiple agents simultaneously
- Shared files → falls back to serial, shared-file tasks run last

---

## Full Workflow

```
You describe a feature (3+ files)
  └→ designer agent writes design doc first
       ├─ Tech spec (data model + API design)
       └─ UI/UX four states (loading / empty / success / error)

You start coding
  └→ Auto-loads language conventions (write correctly from the start)
  └→ Auto-checks cross-layer impact
  └→ Independent tasks run in parallel automatically

Build fails / test errors / unexpected behavior
  └→ debugger agent: root cause → minimal fix → prevention tips

After coding
  └→ reviewer agent loads conventions by tech stack, reviews and fixes directly
       ├─ Critical/Warning → fixed automatically
       └─ Suggestion → reported only, not auto-fixed
```

---

## Parallel Execution

| Scenario | Mode |
|----------|------|
| Backend module A + Backend module B (no file overlap) | Parallel ✅ |
| Frontend page A + Frontend page B (no file overlap) | Parallel ✅ |
| Backend API + Frontend page (after API contract is set) | Parallel ✅ |
| Multiple tasks sharing the same file (e.g. gateway.go) | Serial ⚠️ |
| Starting to code before design doc is done | Serial ⚠️ |

Verified: `tag.go` (22s) + `Tags.vue` (67s) launched simultaneously, total 67s vs 89s serial — ~25% faster.

---

## Installation

**macOS / Linux:**
```bash
git clone git@github.com:martin98ksJ/claude-strengthen-workflow.git
cd claude-strengthen-workflow
bash install.sh
```

**Windows (PowerShell):**
```powershell
git clone git@github.com:martin98ksJ/claude-strengthen-workflow.git
cd claude-strengthen-workflow
.\install.ps1
```

Restart Claude Code to take effect.

> The install script automatically backs up existing `~/.claude/` files (agents/, skills/, CLAUDE.md) to `~/.claude/.backup/<timestamp>/` before overwriting. Existing agents and skills with different content will prompt before overwriting.

---

## Verify Installation

| Test | Expected behavior |
|------|------------------|
| Say "write a Go API endpoint" | Reads go-conventions + backend-conventions before coding |
| Say "add a new feature involving multiple files" | Triggers designer agent to write design doc first |
| Say "check my changes" | Triggers reviewer agent to review and fix |
| Build or test fails | Triggers debugger agent to locate and fix root cause |
| Give two independent tasks at once | Detects file overlap; runs in parallel if no overlap |

---

## Uninstall

**macOS / Linux:**
```bash
bash uninstall.sh
```

**Windows (PowerShell):**
```powershell
.\uninstall.ps1
```
