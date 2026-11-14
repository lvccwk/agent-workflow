# Stored Procedure CR — Controlled Agent Rules

## Mission
Work on ONE Stored Procedure Change Request (CR) at a time. Target correctness, minimal SQL changes, traceable evidence, and a short human-readable report. These rules are instructions, not a substitute for tool permissions, database privileges, or human approval.

## Scope and folders
- `CR/`: approved business requirements and acceptance criteria; READ ONLY.
- `DATABASE/`: reference DDL, other procedures, and schema; READ ONLY.
- `OLD/`: original source code, immutable baseline; READ ONLY.
- `SOLUTION/`: only approved changed SQL source code.
- `REPORT/`: short phase summaries, matrices, review findings.
- `TEST/verification_scripts/`: proposed test SQL; MUST NOT be executed automatically.
- `TEST/expected_results/`: approved expected outputs, if provided; do not rewrite based on actual output.
- `TEST/actual_results/`: actual evidence supplied by authorized testers; READ ONLY.
- Never write to any location outside `SOLUTION/`, `REPORT/` and permitted parts of `TEST/`.

## Database and SQL rules
- Determine SQL dialect (SQL Server / Oracle / DB2 / PostgreSQL / other) from evidence. If uncertain, label UNKNOWN and ask. Do not assume SQL Server.
- Never connect to DEV, SIT, UAT or PROD databases; never run SQL, stored procedures, migrations, deployment, or data fixes.
- Never inspect secrets or customer personal data unless explicitly authorized and unavoidable. Never upload internal source or diagrams to external services unless organization-approved.
- The user's authorized DB operator runs approved tests in a controlled environment and supplies redacted results.
- Explicitly flag any verification SQL that writes data or calls a procedure with side effects. Never describe it as read-only.
- Preserve parameter signatures, output schema, transactions, error handling, and unrelated logic unless approved requirement explicitly changes them.

## Eight phases; no skipping
01 UNDERSTAND: Read target SP and confirmed dependencies. Describe AS-IS only.
02 REQUIREMENTS: Requirement ID -> AS-IS -> TO-BE -> acceptance rule. Stop on material ambiguity.
03 IMPACT: Affected SP, tables, callers, jobs, outputs, side effects, risk, missing dependencies.
04 PLAN: Minimal-diff change plan for ONE requirement; tests and rollback considerations. STOP at Gate A.
05 BUILD: Only after explicit user approval. Implement ONE Requirement ID in `SOLUTION/` only, then STOP.
06 REVIEW: Compare baseline and solution independently; no automatic repair; label static vs executed evidence.
07 UAT DESIGN: Test matrix with independent expected outcomes and controlled verification SQL drafts. STOP at Gate B before actual UAT.
08 UAT ASSESSMENT: Assess actual tester-supplied evidence; propose status, never sign off for business.

## Human approval gates
- Gate A before editing SQL: require explicit human message `APPROVE BUILD <Requirement-ID>` for the precise change plan. A generated plan/report is NOT approval.
- Gate B before executing UAT: actual tests may proceed only after human authorization via organization process, with deployed version, dataset and test plan confirmed. The agent never executes them.
- Gate C for business UAT sign-off: only named business owner or authorized approver can sign. Agent gives recommendation only.
- A slash command does NOT grant approval on its own. A model may not approve itself or infer approval from previous messages.
- If Gate status is unclear: STOP, status BLOCKED, state what needs approval.

## Anti-hallucination and evidence
- Every business-rule claim needs a source path plus relevant line range or SQL excerpt.
- Each proposed change must have Requirement ID, original SQL, proposed SQL, reason, impacted dataset, and regression check.
- `CONFIRMED` = visible in supplied source or business acceptance document; `INFERRED` = plausible but unverified; `UNKNOWN` = missing evidence. Do not promote inferred rules to confirmed.
- Treat dynamic SQL, synonym/view, scheduler job, trigger, external call, table schema and production data assumptions as UNKNOWN unless shown.
- Expected UAT outputs must be derived from business rules or approved fixtures, NEVER copied from the modified procedure's output.
- A static review is NOT a syntax compilation, SQL execution, performance result, or UAT PASS.
- A test may be PASS only if evidence identifies tested version, dataset, execution context, expected and actual results.

## Minimal change / stop rules
- One requirement and one SP at a time, unless its dependency cannot be changed independently (explain and request permission).
- No unrelated cleanup, formatting, refactoring, renaming, restructuring, or speculative improvements.
- No silent repair loops. A failure gets one diagnosis; at most two proposed correction attempts **after human approval for each change**. Then STOP and report blocker.
- If SQL dialect, business rule, schema, dependency, baseline, test dataset, or approval is missing and affects correctness: STOP.
- Never run a tool to bypass read-only boundaries or OpenCode permissions.
- Any mismatch between plan and SQL diff: STOP, report deviation, request a new approval.

## Short-report contract (mandatory)
- **No essay**, no generic preface, no repeated narration, no praise, no multi-page conclusions.
- Conversation response: maximum 12 short lines, unless user explicitly requests a deep dive.
- Phase report top section: maximum ~35 lines. Additional evidence can live in `REPORT/appendix_<phase>.md` only if really necessary.
- Use one compact matrix (max 10 rows in main report); mark additional items `+N more, see appendix`.
- Use the exact template below, then STOP:

  `Phase: 03 IMPACT | Scope: R01 / sp_name | Status: COMPLETED/BLOCKED/INCOMPLETE`

  `| Object / SQL evidence | Change or risk | Action |`

  `| ... | ... | ... |`

  `Critical gaps: <up to 3 concise items, or None>`

  `Decision required: <one clear yes/no or specific missing fact>`

  `Next step: <one command only>`

- Put code or SQL snippets only where needed to prove a rule or illustrate a change; no large source-code dumps.
- Respond in Traditional Chinese using precise SQL identifiers in English.

## Phase artifacts
- `REPORT/01_as_is.md`
- `REPORT/02_requirement_matrix.md`
- `REPORT/03_impact.md`
- `REPORT/04_plan_<Requirement-ID>.md`
- `REPORT/05_diff_<Requirement-ID>.md`
- `REPORT/06_review_<Requirement-ID>.md`
- `REPORT/07_uat_matrix.md`
- `REPORT/08_uat_assessment.md`
- Generated files must include `NOT RUN` for all tests without execution evidence.
