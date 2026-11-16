# Agent Workflow — OpenCode CR Agent

A documentation and knowledge collection for Change Request (CR) engineering workflows, focused on **Java / SQL Server Stored Procedures / Insurance Agency Reporting** changes.

Design principle: the AI Agent produces **auditable deliverables**; business requirement approval, environment operations, UAT sign-off and production deployment remain the responsibility of authorised personnel.

---

## 📁 Repository Structure

```
.
├── docs/         Rules, guidelines and project structure definitions
├── guides/       01–07 end-to-end operating manual (Markdown + Word)
├── prompts/      Ready-to-copy prompts
├── knowledge/    Phone capture inbox (auto-categorised archive)
└── README.md
```

---

## 📚 Document Index

### docs/ — Rules and Guidelines

| File | Description |
|---|---|
| [AGENTS.md](docs/AGENTS.md) | **Core rules**: agent mission, folder contract, safety rules, mandatory workflow and stop points, quality gates, response format |
| [CR-Development-Instructions.md](docs/CR-Development-Instructions.md) | Condensed development guide: folder rules + six-step workflow |
| [CR-001-structure.md](docs/CR-001-structure.md) | CR-001 standard folder structure example (CR / DATABASE / OLD / SOLUTION / TEST / REPORT) |
| [SQL_Developer/AGENTS.md](docs/SQL_Developer/AGENTS.md) | **Stored Procedure CR controlled-agent rules**: eight-phase workflow, Gate A/B/C human approvals, CONFIRMED/INFERRED/UNKNOWN evidence grading, minimal diff, short-report contract |
| [SQL_Developer/opencode.jsonc](docs/SQL_Developer/opencode.jsonc) | **OpenCode strict permission baseline**: edit limited to REPORT/ (SOLUTION/ requires ask), shell only git status/diff, webfetch/websearch/subagent all denied |
| [SQL_Developer/SDLC-Framework.md](docs/SQL_Developer/SDLC-Framework.md) | **SP Change Request SDLC framework**: 7-phase workflow (Mermaid), role responsibility matrix, risks and rollback, Git version control, report writing guide |
| [SQL_Developer/AGENTS_9Step.md](docs/SQL_Developer/AGENTS_9Step.md) | **9-step condensed rule set** — copy straight into `CR-001/AGENTS.md`. Role, folder access, safety floor, 9-step / 3-gate workflow, evidence grading, minimal-change rule, stop rules, fixed 12-line reply format |
| [SQL_Developer/AGENTS_9Step_Notes.md](docs/SQL_Developer/AGENTS_9Step_Notes.md) | **Why the 9-step rules are written that way** — section-by-section rationale, English-rules / Chinese-replies decision, and the two-layer defence (`AGENTS.md` advises, `opencode.jsonc` enforces) |

### guides/ — Complete Operating Manual

| File | Description |
|---|---|
| [OpenCode_CR_Agent_01-07.md](guides/OpenCode_CR_Agent_01-07.md) | **01–07 end-to-end manual**: per-phase purpose, actions, prompts, deliverables, stop conditions; appendices include the full AGENTS.md |
| [OpenCode_CR_Agent_01-07.docx](guides/OpenCode_CR_Agent_01-07.docx) | Word version of the same manual (for distribution/printing) |
| [CR_9Step_Prompt_Flow.md](guides/CR_9Step_Prompt_Flow.md) | **9 steps / 3 gates quick playbook**: one copy-ready prompt per step (folder → as-is → requirements → impact → plan → build → review → UAT → report), fixed guard-rail rules, Gate A/B/C stop points |

**Phase mapping:**

| Step | Phase | Core output |
|---|---|---|
| 01 | Requirement Intake (Phase 0) | Requirement Register |
| 02 | As-Is & Impact (Phase 1 + 2) | As-Is / Dependencies / Impact |
| 03 | To-Be & Plan (Gate A) | Implementation Plan (awaiting approval) |
| 04 | Build | SOLUTION/ changes + Diff |
| 05 | Independent Verification | Code Review Report |
| 06 | UAT Preparation & Evidence | Test cases + actual execution evidence |
| 07 | Release Readiness | Release review |

### prompts/ — Ready-to-use Prompts

| File | Purpose |
|---|---|
| [START_CR_PROMPT.txt](prompts/START_CR_PROMPT.txt) | Start or resume a CR (continue from the current phase; never skip a gate) |
| [phase5-independent-verification.md](prompts/phase5-independent-verification.md) | Phase 5 independent verification: Prompt A (code review) + Prompt B (test case generation) |
| [session-review.md](prompts/session-review.md) | Session handover: produce `REPORT/session_handover.md` |

### knowledge/ — Phone Capture Archive

| File | Description |
|---|---|
| [INDEX.md](knowledge/INDEX.md) | Master index (auto-maintained; one line added per saved item) |
| [ai/2026-11-14-opencode-sql-agent-improvement.md](knowledge/ai/2026-11-14-opencode-sql-agent-improvement.md) | **Agent improvement recommendations**: evidence-driven development, target-SP scoping, minimal diff, retry limit, reviewer agent separation |

Category folders: `inbox/` (uncategorised), `ai/`, `dev/`, `finance/`, `life/`, `reading/`

---

## 🚀 Quick Start

1. Copy `AGENTS.md` from `docs/` into your CR working directory (OpenCode reads it automatically).
2. Use `prompts/START_CR_PROMPT.txt` to start each session.
3. After each phase, run the corresponding verification prompt in `prompts/` as an independent check.
4. Label every conclusion as `Verified` / `Assumed` / `Unknown`, with file paths and line numbers.

---

## ⚠️ Core Safety Rules (Summary)

- **Never** execute production SQL. **Never** deploy automatically.
- `CR/`, `DATABASE/`, `OLD/` are **read-only**.
- `SOLUTION/` is modified **only after approval**.
- Any inference must be labelled `Assumed` or `Unknown`; never present as fact.
- Automatic correction is limited to **2 attempts**, after which work must stop and human review must be requested.

---

*Last updated: 2026-11-16*
