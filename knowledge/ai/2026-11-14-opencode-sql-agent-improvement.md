---
title: OpenCode SQL Agent Improvement — Evidence-Driven Development
source: telegram
saved: 2026-11-14
category: ai
tags: [agent, opencode, sql, evidence-driven, workflow]
note: Agent improvement recommendations sent by the user via Telegram
---

# OpenCode SQL Agent Improvement — Evidence-Driven Development

> Goal: eliminate the agent's repeated mistakes, re-analysis and incorrect Stored Procedure changes.
> Core shift: from "ask the agent to complete the work" to "ask the agent to provide evidence that every step is correct".

---

## 1. Why does the agent keep getting it wrong?

| Problem | Likely cause | Improvement |
|---|---|---|
| Misunderstands business logic | Agent infers requirements on its own | Require cited SQL evidence |
| Modifies the wrong Stored Procedure | Analysis scope too broad | Specify the target SP |
| Changes unrelated SQL | No explicit change boundary | Enforce minimal diff |
| Re-analyses repeatedly | Session context too long | Split into staged sessions |
| Inaccurate UAT SQL | Agent assumes expected results | Confirm business rules first |
| Endless retries | No stop condition | Set a retry limit |

**The single most important change**: from "ask the agent to complete the work" to "ask the agent to provide evidence that every step is correct".

---

## 2. First improvement: do not analyse the whole project at once

Previously used prompt:

```text
Analyze this project and explain:
System architecture and data flow
Key business logic
Stored procedure dependencies
Potential risks and edge cases
```

**Problem**: the scope is too broad. If `DATABASE/` contains 20 stored procedures, the agent may spend most of its time on irrelevant SQL, or confuse the business purpose of different procedures.

### Recommended replacement — Prompt 1.1

```text
Act as a Senior SQL Server System Analyst.
Your task is to understand the EXISTING
implementation of this Change Request.

Scope:
CR/ : Business requirements
OLD/ : Original stored procedures
DATABASE/ : Reference procedures and schema

Instructions:
Identify the target stored procedure(s)
based on the CR requirements.
Analyze ONLY the target procedures
and their confirmed dependencies.
Explain the existing logic step by step.
Identify:
Input parameters
Source tables
JOIN conditions
WHERE conditions
CASE WHEN logic
Temporary tables
Output tables
For each business rule, provide:
SQL file
Procedure name
Relevant line number
Explanation
Confidence level
Mark any unverified business rule
as UNKNOWN.

Do not propose implementation changes.
Do not modify files or execute SQL.
STOP after the As-Is analysis.

Explain in English.
```

**Point 5 matters most** — require:

```
Business Rule → SQL Statement → File Reference
```

For example:

| Business Rule | SQL Evidence | Status |
|---|---|---|
| Title Code >= 100 | sp_Agency.sql Line 145 | Verified |
| Inforce Agency Head | sp_Agency.sql Line 178 | Verified |
| Reporting Relationship | Missing reference | Unknown |

This forces the agent to separate **verified code behaviour** from **its own inferred business interpretation**.

---

## 3. Second improvement: adopt Evidence-Driven Development

Add the following rules to `AGENTS.md`:

### Evidence-Driven SQL Development Rules

#### 1. No Assumptions

- Never invent business rules.
- If a requirement is unclear:
  - Identify the ambiguity.
  - Explain why it matters.
  - Ask for clarification.
  - Stop implementation if the ambiguity affects correctness.

#### 2. Evidence Required

Every proposed SQL modification must include:

- Requirement ID
- Procedure name
- Original SQL statement
- Proposed SQL statement
- Reason for modification
- Potential side effects

#### 3. Minimal Changes

Only modify SQL statements required by approved requirements.

Do not:

- Refactor unrelated SQL
- Rename variables unnecessarily
- Change existing calculations
- Modify unrelated JOIN conditions
- Change output schema without approval

#### 4. Validation

After implementation:

- Compare OLD vs SOLUTION.
- Identify all SQL changes.
- Map every change to requirements.
- Check for unintended changes.
- Identify unverified behavior.

#### 5. Stop Conditions

STOP immediately if:

- Business requirements conflict.
- Required SQL dependencies are missing.
- A proposed change affects unknown logic.
- Tests fail repeatedly.
- The approved implementation plan cannot be followed.

Do not automatically attempt unlimited fixes.

**Maximum automatic correction attempts: 2.**

After 2 failed attempts:

- Stop.
- Report the error.
- Explain possible root causes.
- Request human review.

#### 6. Safety

- Never execute production SQL.
- Never deploy changes automatically.
- Never modify `OLD/` or `DATABASE/`.
- All explanations must be in English.

> This is a prompt-level control, not an absolute guarantee. It should be combined with OpenCode Permissions to restrict file modification and shell execution.

---

## 4. Third improvement: deliver one small task at a time

For example, three stored procedures need to be modified.

**❌ Not recommended:**

```text
Implement all CR changes and verify
that everything is correct.
```

**✅ Recommended:**

```text
Implement Requirement R01 only.

Target:
SOLUTION/sp_AgencyReport.sql

Approved change:
Update the agency head eligibility logic
according to the implementation plan.

Restrictions:
Modify only this stored procedure.
Preserve all unrelated logic.
Do not change output columns.
Do not modify other files.
Do not execute SQL.

After modification:
Show the exact SQL diff.
Explain each changed statement.
Map changes to Requirement R01.
Identify regression risks.
Stop and wait for review.
```

Complete R01 → review the diff → then proceed to R02.

This approach surfaces problems far more easily than modifying all SPs in one pass.

---

## 5. Fourth improvement: add an independent Reviewer Agent

OpenCode supports dedicated subagents (for example a Reviewer) that run in a separate context to perform code review.

Work can be split as follows:

| Agent | Responsibility |
|---|---|
| Analyst | Analyse requirements and the As-Is state |
| Developer | Modify SQL according to the approved plan |
| Reviewer | Independently compare CR, OLD and SOLUTION |
| QA | Generate UAT cases and check actual results |

Four custom agents are not mandatory. Initially, separate sessions with separate prompts are sufficient.

However, the same model may repeat the same mistakes even in a new session, so **the reviewer must have clear verification criteria**.

---

## 6. How should GPT-6 Luna High be used?

Try the following arrangement first rather than immediately switching everything to a more expensive model.

| SDLC activity | Recommendation |
|---|---|
| As-Is Analysis | Luna High + restricted target SP |
| Requirement Analysis | Luna High + evidence matrix |
| Impact Analysis | Luna High + dependency list |
| Implementation Plan | Luna High + human approval |
| SQL Modification | Luna High + one requirement at a time |
| Independent Review | New session, switch model if necessary |
| UAT Test Generation | Based on confirmed business rules |
| UAT Acceptance | Actual execution evidence + human sign-off |

> High reasoning provides more inference budget, but does not automatically supply missing schema, business rules or execution results.

---

## 7. Identifying the core problem

When Luna High "keeps producing inaccurate prompts", identify which type of failure applies:

| Type | Description |
|---|---|
| **A. Comprehension problem** | The agent frequently misreads SQL business logic when analysing stored procedures |
| **B. Execution problem** | The agent analyses correctly but modifies the wrong location or misses conditions |
| **C. Verification problem** | The agent claims the change is fine, but manual review finds errors in the SQL or UAT test cases |
| **D. Workflow problem** | The agent re-analyses and re-modifies repeatedly, requiring many loops to finish one issue |

- If **A** → redesign the analysis prompt
- If **B or C** → strengthen diff verification and test evidence
- If **D** → restructure the overall OpenCode agent loop

---

*Compiled: 2026-11-14*
