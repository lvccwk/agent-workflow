# Stored Procedure CR - Lite Agent Rules

## Mission and setup

Analyse ONE CR at a time using the three steps in `SP_CR_Workflow_Lite.md`:
1. Analyse changes and impacted reports.
2. Produce the business report and test drafts.
3. Assess DBA-supplied results.

This workflow compares existing original and modified SPs. It does not build or repair application SQL. A CR may include multiple SPs; keep each change traceable to its SP and requirement.

Follow `README.md` for initial folder setup. Copy this file to `CR-<ID>/AGENTS.md`, the project configuration to `CR-<ID>/opencode.jsonc`, and `CR_Report_Template_Business.md` to `CR-<ID>/REPORT/`. Open `CR-<ID>/` as the working folder; all paths below are relative to that root. Keep the shared workflow and README available as reference documents.

These instructions do not grant tool permissions, database privileges or business approval. Respect actual tool permissions; never bypass a denied operation through another tool or shell command.

## Inputs and write boundaries

| Path | Purpose | Agent access during the workflow |
|---|---|---|
| `CR/CR.txt`, `CR/CR_attachment.csv` | Approved requirements and acceptance criteria | Read only |
| `CR/report_list.csv` | Credential-free report inventory | Read only |
| `transition_plan.txt ` | Transition requirements | Read only |
| `DATABASE/` | Reference DDL, SPs and callers | Read only |
| `OLD/` | Original SPs; immutable baseline | Read only |
| `SOLUTION/` | Modified SPs supplied for comparison | Read only |
| `REPORT/CR_Report_Template_Business.md` | Business report template | Read only |
| `REPORT/` generated files | Analysis, business report and test cases | Create/update |
| `TEST/uat_queries.sql`, `TEST/verification_scripts/` | Test SQL drafts | Create/update when tool permissions allow; never execute |
| `TEST/expected_results/` | Approved expected outputs, if supplied | Read only |
| `TEST/actual_results/before/`, `TEST/actual_results/after/` | Redacted evidence supplied by the DBA | Read only |

Do not change inputs, configuration or files outside these output paths during a CR analysis. Setup and maintenance of these shared templates are separate from the CR workflow and may be performed when explicitly requested by the user.

Before Step 1, check that inputs and corresponding SQL files in `OLD/` and `SOLUTION/` exist. If no modified SPs are supplied, request them; do not create a solution as part of this comparison workflow. Identify the SQL dialect from supplied evidence; use `UNKNOWN` when it cannot be established.

## Step 1 - Analyse

- From the CR root, run `git diff --no-index -w OLD/ SOLUTION/ > REPORT/00_raw_diff.txt` if permitted. Git exit code 1 means differences were found; errors must be investigated before using the output. Do not overwrite existing evidence without a user request to regenerate it.
- Create `REPORT/01_changes.md` using `CR/` and `transition_plan.txt `. Use one row per diff hunk, with stable C-IDs:

  `| C-ID | SP | Line | Before | After | REQ-ID | CR quote |`

- Cite source paths and line references. Identify original and modified line locations where relevant. Map all requirements and list both changes without a requirement and requirements without a change. Explain file additions/deletions and any known changes excluded by whitespace ignoring; do not silently omit them.
- Add an **Impacted User Reports** section to the same file. Use `CR/report_list.csv` and supplied `DATABASE/` references; trace at most two caller levels:

  `| R-ID | Excel Name | Connection Name | Command Text | Used In | Changed Object Used (C-ID) | Business Effect |`

- Assign stable R-IDs to reports; use REQ-IDs for requirements to avoid confusing the two. Copy credential-free Command Text exactly from the inventory. If credentials are present, do not reproduce them; request a sanitized input.
- State the number of reports checked and impacted. Support each dependency and business effect with evidence. Missing caller definitions, dynamic SQL or unresolved dependencies are `UNKNOWN`, not proof that a report is unaffected. Mark the two-level tracing limit where deeper dependencies may exist.

**Checkpoint 1:** Stop after completing Step 1 for review of diff coverage, requirement mapping and impacted reports. Continue to Step 2 when the user authorizes it. Do not require a separate BUILD approval for read-only comparison and report drafting.

## Step 2 - Report and tests

Use the reviewed `REPORT/01_changes.md`, baseline, modified SPs and references.

### Business report

- Read `REPORT/CR_Report_Template_Business.md` and save the completed report as `REPORT/CR_Report.md`. Preserve the template file.
- Use formal business English without emojis. Retain the template's sections 1-10 and replace sample SPs, rules, counts, test results and evidence paths with actual findings. Template examples are not evidence.
- Under section 6, use **6.1 Object Impact** for the existing object analysis and add **6.2 Impacted User Reports**. Copy the reviewed impacted-report table and show the supported path: changed SP -> output table -> Excel report (Used In).
- Build Before/After architecture and SP data flow diagrams from supplied source. Limit each Mermaid diagram to 10 nodes; split diagrams if necessary. Use red shading for changed nodes and green for new nodes. Set `color:#000000` on shaded nodes, including sections 3.2, 4.2 and 6, to keep labels readable.
- Keep unsupported details `UNKNOWN`. Explain risks and a source-backed rollback proposal; do not deploy or execute it.
- Until evidence is supplied, set all test results to `NOT RUN`, leave actual values pending and mark evidence as not supplied. Replace the template's sample PASS values and pie-chart counts. Do not label the CR UAT Passed or fill in sign-off on someone else's behalf.

### Test cases

Create `REPORT/test_cases.md` with two parts and unique Test-IDs:

**A) Logic tests** - normal, boundary, NULL and regression scenarios:

`| Test-ID | C-ID | Scenario | Expected (per CR) | Before | After | Result | Evidence |`

**B) User report tests** - at least one case for every impacted Excel report:

`| Test-ID | R-ID | Excel Name | Used In | Expected change (per CR) | Before | After | Result | Evidence |`

Derive expected outcomes from approved requirements or independent approved fixtures, never from modified-SP output. If the expected outcome is missing, mark it `UNKNOWN` and identify the acceptance decision required. Initial Result is `NOT RUN`.

### Test SQL drafts

- Create `TEST/uat_queries.sql` with A) Logic tests and B) User report tests. Additional drafts may go in `TEST/verification_scripts/`.
- Confirm dialect and schema before drafting executable SQL. Logic checks use read-only SELECTs with COUNT, SUM on confirmed numeric columns and a dialect-appropriate limit of 10 rows. Select only approved, non-sensitive columns; do not use SELECT * on customer data.
- For each impacted R-ID, include the original sanitized Command Text and header comments with Test-ID, Excel Name, Connection Name and Used In. Add COUNT/SUM wrappers only where the query syntax and result columns support them. Do not wrap EXEC or multi-statement commands as derived tables.
- Keep EXEC, DML, DDL, SELECT INTO and commands with unknown side effects in comments, labelled `EXEC - DBA confirmation required` or an appropriate side-effect warning. Do not describe these commands as read-only. Ask the DBA for a reviewed execution method or a read-only alternative.
- Specify identical datasets, parameters and reconciliation rules for before/after comparison. The DBA supplies `TEST/actual_results/before/<Test-ID>.csv` and `TEST/actual_results/after/<Test-ID>.csv`, with tested version, environment, execution time and dataset/parameter context.
- If configuration denies `TEST/uat_queries.sql`, report the exact blocked path and request an authorized configuration adjustment. Do not relocate the deliverable silently or write it using shell redirection to bypass the restriction.

**Checkpoint 2:** Stop after report and test drafting. The reviewer/DBA confirms query behavior and execution authorization. Only the authorized DBA runs before-deployment tests, deploys, and runs after-deployment tests. The agent never runs SQL or connects to a database.

## Step 3 - Results

- Read DBA-supplied evidence in `TEST/actual_results/before/` and `TEST/actual_results/after/`; do not create or alter execution evidence.
- Update `REPORT/test_cases.md` and section 7 of `REPORT/CR_Report.md`. PASS requires a known tested version, dataset, execution context, expected outcome and matching actual results. Missing evidence remains `NOT RUN`; missing acceptance rules remain `UNKNOWN` and cannot produce PASS.
- Cite exact evidence paths. Explain each FAIL with its C-ID and, where relevant, R-ID. Reconcile case totals, summary metrics and chart counts with the test table.
- Update the executive summary and recommend `GO`, `NO-GO` or `CONDITIONAL`, with supporting reasons and unresolved conditions. This is a recommendation, not deployment authorization or business sign-off.

**Checkpoint 3:** Stop for report-owner validation and Business Owner sign-off. Leave signature fields blank for the authorized people.

## Evidence, security and communication

- `CONFIRMED` means supported by supplied source or approved business evidence; `INFERRED` means plausible but unverified; `UNKNOWN` means missing evidence. Keep these distinct.
- Never connect to DEV, SIT, UAT or PROD or execute SQL, procedures, migrations or deployments in this workflow. Execution belongs to the authorized DBA.
- Do not inspect secrets or customer personal data unless explicitly authorized and unavoidable. Never upload internal source or diagrams to external services without explicit organizational authorization.
- Never invent requirements, dependency links, business effects, benchmark results or UAT evidence. Static review is not compilation, execution or UAT PASS.
- Flag SQL injection, unsafe dynamic SQL, exposed credentials or other observed security issues with source evidence; do not silently repair input SQL or claim a full security audit or vulnerability-free dependencies without verification.
- If an input or permission is missing, continue work that is independent of it, mark affected findings incomplete and identify the exact missing item. Do not mark the checkpoint complete while material gaps remain.
- Keep conversation replies concise and in Traditional Chinese. Reports use formal business English. Avoid generic essays, duplicate phase files and unrelated refactoring. Do not truncate the required change/test matrices to meet a conversation-length limit.
- Use `Step: 1/2/3 | Status: COMPLETED/INCOMPLETE/BLOCKED`, followed by outputs, material gaps and the next checkpoint decision. Do not introduce the old eight-phase workflow or require its phase artifacts.

## Required deliverables

| File | Step |
|---|---|
| `REPORT/00_raw_diff.txt` | 1 |
| `REPORT/01_changes.md` | 1 |
| `REPORT/CR_Report.md` | 2, 3 |
| `REPORT/test_cases.md` | 2, 3 |
| `TEST/uat_queries.sql` | 2 |
| `TEST/actual_results/before/<Test-ID>.csv`, `TEST/actual_results/after/<Test-ID>.csv` | DBA supplies; agent reads in 3 |
