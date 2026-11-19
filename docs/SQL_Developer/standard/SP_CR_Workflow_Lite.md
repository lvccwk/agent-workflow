# Stored Procedure CR Workflow (Lite)

Three prompts, three checkpoints. Review each output before proceeding.

```mermaid
flowchart LR
    P["Prepare<br/>Inputs"] --> A["Step 1<br/>Analyse"]
    A --> C1{"Review<br/>Changes and Impact"}
    C1 --> B["Step 2<br/>Report and Tests"]
    B --> C2{"Review SQL<br/>DBA Before / After"}
    C2 --> R["Step 3<br/>Results"]
    R --> C3{"Business<br/>Sign-off"}

    classDef step fill:#eef2f7,stroke:#44546a,color:#1f2937
    classDef gate fill:#ffffff,stroke:#1f3864,stroke-width:2px,color:#1f3864
    class P,A,B,R step
    class C1,C2,C3 gate
```

## Prepare

| Input | Save as |
|---|---|
| CR Word document and Excel attachment | CR/CR.txt, CR/CR_attachment.csv |
| Report inventory (credentials removed) | CR/report_list.csv |
| NSC transition plan | NSC_transition_plan.txt |
| Report template | REPORT/CR_Report_Template_Business.md |

## Step 1 — Analyse

1. Run: `git diff --no-index -w OLD/ SOLUTION/ > REPORT/00_raw_diff.txt`
2. Using CR/ and NSC_transition_plan.txt, create REPORT/01_changes.md:

   | C-ID | SP | Line | Before | After | REQ-ID | CR quote |
   |---|---|---|---|---|---|---|

   One row per diff hunk. List changes without a requirement and
   requirements without a change.
3. Using CR/report_list.csv and DATABASE/ (max 2 levels of callers),
   add a section "Impacted User Reports". One row per Excel report
   whose Command Text uses a changed SP or a table it writes to:

   | R-ID | Excel Name | Connection Name | Command Text | Used In | Changed Object Used (C-ID) | Business Effect |
   |---|---|---|---|---|---|---|

   Copy Command Text exactly as in report_list. State the number
   of reports checked and the number impacted.

Unsure = UNKNOWN. Do not modify OLD/ or SOLUTION/.

**Checkpoint 1:** All hunks captured, all requirements mapped, impacted reports confirmed.

## Step 2 — Report and Tests

Using REPORT/01_changes.md, OLD/, SOLUTION/ and DATABASE/:

1. Complete REPORT/CR_Report_Template_Business.md and save as
   REPORT/CR_Report.md. Formal business English, no emojis.
   Mermaid diagrams from the actual SPs, max 10 nodes, changes
   highlighted. Test results = NOT RUN.
   Section 6.2: copy the "Impacted User Reports" table from
   01_changes.md, and draw one diagram:
   changed SP -> output table -> Excel report (Used In).
2. Create REPORT/test_cases.md with two parts:

   **A) Logic tests**

   | Test-ID | C-ID | Scenario | Expected (per CR) | Before | After | Result | Evidence |
   |---|---|---|---|---|---|---|---|

   Cover normal, boundary, NULL and regression.

   **B) User report tests, one per impacted Excel report**

   | Test-ID | R-ID | Excel Name | Used In | Expected change (per CR) | Before | After | Result | Evidence |
   |---|---|---|---|---|---|---|---|---|
3. Create TEST/uat_queries.sql with two sections:

   **A) Logic tests:** SELECT only; per Test-ID COUNT, SUM, top 10 rows.

   **B) User report tests:** for each R-ID, the report's original
   Command Text, followed by a COUNT and SUM wrapper of it.
   Header comment per query: Excel Name, Connection Name,
   Used In. Label any EXEC "EXEC - DBA confirmation required".
   Run before and after deployment with identical data and
   parameters; save to actual_results/before/ and
   actual_results/after/ as <Test-ID>.csv.

**Checkpoint 2:** Queries confirmed read-only. DBA executes before deployment, deploys, executes again.

## Step 3 — Results

Read TEST/actual_results/before/ and after/. Update REPORT/test_cases.md
and section 7 of REPORT/CR_Report.md with PASS/FAIL and evidence.
Explain each FAIL with related C-ID. Update the executive summary and
recommend GO, NO-GO or CONDITIONAL.

**Checkpoint 3:** Report owners validate; Business Owner signs off CR_Report.md.

## Deliverables

| File | Step |
|---|---|
| REPORT/01_changes.md | 1 |
| REPORT/CR_Report.md | 2, 3 |
| REPORT/test_cases.md | 2, 3 |
| TEST/uat_queries.sql | 2 |
| TEST/actual_results/ | DBA |
