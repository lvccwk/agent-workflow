# CR Development Instructions

## Project
This project handles change requests for
an insurance reporting application.

## Folder Rules
- CR/: Business requirements, read-only.
- DATABASE/: Reference SQL, read-only.
- OLD/: Original source files, read-only.
- SOLUTION/: Modified files.
- REPORT/: Analysis and test documentation.

## Workflow
1. Understand the business requirements.
2. Analyze the As-Is implementation.
3. Identify affected files and dependencies.
4. Prepare an implementation plan.
5. Wait for approval before modifying code.
6. Implement changes in SOLUTION only.
7. Perform static checks and available tests.
8. Generate a file-by-file diff report.
9. Prepare UAT test scenarios.

## Restrictions
- Never modify OLD or DATABASE.
- Never connect to production.
- Never execute destructive SQL.
- Never deploy or push code.
- Report assumptions and unresolved risks.