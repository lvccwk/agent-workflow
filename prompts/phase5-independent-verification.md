Prompt A — Independent Code Review
Act as an independent Senior System Analyst and SQL Code Reviewer.

Review CR/requirement.md, DATABASE/, OLD/, SOLUTION/ and REPORT/implementation_plan.md.

Tasks:
1. Compare OLD and SOLUTION file by file.
2. Map each change to an explicit business requirement.
3. Verify existing business logic is preserved except for approved changes.
4. Identify missing dependencies, unexpected changes, NULL handling, JOIN duplication, boundary conditions and SQL performance risks.
5. Identify any unintended changes to existing reporting results.
6. Classify findings as Critical, High, Medium or Low.
7. Provide file references, evidence, and recommended actions.
8. Mark every claim as verified, assumption, or not tested.

Do not modify files.
Do not execute SQL or access databases.
Provide findings in Traditional Chinese.

Prompt B — Generate Test Cases
Generate a comprehensive Verification and Regression Test Pack for this CR.

Use CR/requirement.md, DATABASE/, OLD/, SOLUTION/ and the implementation plan as references.

Create test cases covering:
1. Positive and negative business scenarios
2. Boundary values and NULL handling
3. Stored procedure dependencies and failure handling
4. Data reconciliation between old and new logic
5. Duplicate, missing and unexpected records
6. Regression testing for unaffected business logic
7. Batch retry, rerun and idempotency where relevant
8. Performance and security considerations

For each case, include: Test ID, Requirement ID, Preconditions, Input Data, Test Steps, Independently Derived Expected Result, Actual Result (Not Run), Status (Not Run), and Evidence Required.

Generate safe, read-only comparison SQL where feasible. Identify any test that requires a controlled database write or execution and request explicit approval instead of running it.

Only create files under TEST/ and REPORT/. Do not modify source code. Do not connect to production. Do not invent test execution results. Explain in Traditional Chinese.