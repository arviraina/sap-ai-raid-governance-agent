# SAP AI RAID Governance Agent

## Human-reviewed AI reporting for overdue SAP project actions

**Project type:** Personal portfolio prototype using fictional training data  
**Tools:** n8n, Groq, JavaScript, structured AI output, n8n Data Tables, HTML  
**Status:** Working local prototype; initial approval, rejection and invalid-count tests completed

### The problem

Reviewing overdue project actions involves more than summarising a list. A project manager needs to understand which actions need attention, what their potential delivery impact could be, what evidence is missing, and whether a report has been reviewed before distribution.

AI can help draft this analysis, but it can also change supplied facts or present an inference too confidently. This prototype explores a controlled reporting workflow: source-based checks, explicit human review, and a saved decision record before an approved report is generated.

### What I built

I built an n8n workflow that turns a fictional SAP action list into a structured governance report. Although the project is called a RAID Governance Agent, this version focuses on overdue action review and risk commentary. It does not yet manage a complete risks, assumptions, issues and dependencies register.

The workflow filters overdue actions and applies deterministic RED, AMBER and GREEN rules. Groq then drafts a structured executive summary, potential impacts, follow-up recommendations and evidence gaps. JavaScript validates selected facts against the original action data before allowing report generation.

A human reviewer chooses Approve or Reject and supplies a name and optional comments. The workflow saves the decision in an n8n Data Table, restores the report context after the storage operation, and routes the result through a decision gate. Approval produces an HTML file marked as reviewed for distribution. Rejection stops before the approved-report nodes.

### Key design decisions

**Keep factual checks outside the model.** Action names, owners, risk ratings and counts are compared with source data. Overdue days are restored from the source rather than accepted from the AI response.

**Treat generated analysis as a draft.** The prompt asks for possible impacts and missing evidence. A structured format makes selected checks possible, but does not establish that the narrative is correct.

**Record the decision before generating the approved file.** Both outcomes pass through the same audit-storage step. The record includes the decision, reviewer-entered name, comments, timestamps and execution identifiers.

**Preserve context across storage.** The table operation returns the inserted row rather than the original report. A dedicated step matches that row to its review and restores the HTML and form fields before routing continues.

### What I tested

| Scenario | Observed behaviour |
|---|---|
| Fresh approval | Saved an approval audit row and generated the report file through the true branch. |
| Fresh rejection | Saved a rejection audit row with a test comment; the approved-report nodes did not execute. |
| Incorrect AI risk count | A temporary test changed the reported RED count to 999. The validator returned FAIL with Incorrect RED count, and the gate took the false branch. |
| Restore normal operation | Removed the temporary test node, restored the original connection, and confirmed valid input passed again. |

The exported workflow was also reviewed to confirm the audit mappings and restored connections. These are observed prototype results, not a claim of exhaustive testing.

### Outcome and learning

The result is a working demonstration of AI-assisted reporting with source-based validation, a human decision point and persistent review records. The project shows how I combine workflow orchestration, structured model output, JavaScript checks and practical governance controls.

An important lesson was that successful storage does not automatically preserve the data needed downstream. Explicitly restoring the report context kept the approved-report path working while adding the audit trail.

### Current boundaries and next improvements

The prototype uses fictional data and has no live SAP connection or automated stakeholder distribution. Reviewer names are self-reported, and the audit table is editable rather than tamper-proof. Duplicate insert prevention and an immutable link to the exact reviewed report are not yet implemented. No time-saving, commercial-impact or production-readiness claims have been measured.

Next improvements include broader negative tests, retry handling, authenticated review, report-version linkage, and a demonstration using documented sample inputs and expected outputs.

### Suggested portfolio evidence

Pair this case study with the workflow diagram, an approved sample report, audit-table evidence for both decisions, and the invalid-count test result. Keep credentials and private data out of any published materials.
