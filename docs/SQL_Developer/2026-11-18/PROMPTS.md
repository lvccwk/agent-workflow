# CR Prompts

## Step 1 — Analyse

Run `git diff --no-index -w OLD/ SOLUTION/ > REPORT/00_raw_diff.txt`.
Create REPORT/01_analysis.md following AGENTS.md section 6.

## Step 1R — Independent review (new session)

Independent reviewer: check REPORT/01_analysis.md against
00_raw_diff.txt, OLD/, SOLUTION/ and CR/report_list.csv using
AGENTS.md section 6. List errors only. Do not modify files.
Check: reviewer errors + HIGH RISK items.

## Step 2 — Report and tests

Using REPORT/01_analysis.md, create REPORT/CR_Report.md,
REPORT/test_cases.md and TEST/uat_queries.sql following
AGENTS.md section 7.
Check: SQL is SELECT only -> DBA runs before / after deployment.

## Step 3 — Results

Update REPORT/test_cases.md and CR_Report.md section 7 from
TEST/actual_results/. Explain each FAIL. Recommend GO / NO-GO / CONDITIONAL.
Check: report users validate -> Business sign-off.
