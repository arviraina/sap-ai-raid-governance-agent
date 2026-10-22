# SAP AI RAID Governance Agent — setup and workflow notes

Reviewed 22 October 2026 against the supplied **AI Career - Project Action Tracker.json** export and the live tests observed in this chat.

## Purpose and current state

A working training prototype that reviews fictional SAP project actions, classifies overdue items, uses Groq for structured AI analysis, validates selected facts against source data, requires a human review decision, saves that decision in an n8n Data Table, and generates an approved HTML report only on the approval path.

This is a local n8n prototype. It does not connect to a live SAP system or distribute reports to stakeholders. The supplied export contains 21 nodes, has no pinned data, and is marked inactive. No production deployment is claimed.

## Workflow at a glance

Manual trigger → sample action data → overdue filter → deterministic risk classification → PM recommendations → AI RAID Analysis → Validate AI Output → Validation Passed? → draft HTML report → human review form → restore HTML → build approval audit → insert audit row → restore review context → Review Decision Gate → update approved HTML → create HTML file.

Groq Chat Model and Structured Output Parser support the AI analysis node.

The validation gate's false branch and the review gate's rejection branch have no downstream nodes. They stop processing at those points; they do not create an approved report or send a notification.

## Inputs and deterministic rules

The Code in JavaScript node currently creates five fictional actions. The preceding Edit Fields node is not a live input integration: the sample-data code sets its own actions and project name.

The If node selects actions where days_overdue is greater than zero. The current sample yields three overdue actions, two RED and one AMBER.

Risk classification is applied before AI analysis:

- RED: critical priority or at least seven days overdue.
- AMBER: otherwise high priority or at least three days overdue.
- GREEN: otherwise.

These are prototype rules, not a verified SAP delivery-risk methodology. PM Recommendations adds predefined guidance according to the assigned rating.

## AI analysis and validation

The exported Groq node selects `openai/gpt-oss-20b`. This records the export's configuration, not a guarantee of future model availability. The prompt requests possible delivery impacts, practical follow-up, evidence gaps, and an executive summary. A structured output parser defines the expected result shape.

Validate AI Output compares the result with Generate Overdue Report. It checks action counts, missing or duplicated action names, owners, risk ratings, aggregate risk counts, a non-empty summary and impact, and minimum recommendation/evidence-gap array lengths. It restores overdue days from source data.

Validation returns PASS or FAIL plus error messages. Generate HTML RAID Report also refuses data whose validation status is not PASS. The validator does not prove that AI-written impacts are factually correct; human review remains necessary. It does not currently compare the AI project name with the original project name.

## Human review and report handling

Generate HTML RAID Report starts with PM review PENDING and distribution NOT APPROVED. It escapes displayed text before inserting it into HTML.

Human Approval Gate1 checks distribution_approved. The generated draft sets this to false, routing it to Await PM Review. The gate's true branch is unconnected.

The review form collects:

- Submit Review: required dropdown, Approve or Reject.
- Reviewer Name: required text.
- Review Comments: optional text.
- Form metadata includes submission time and form mode.

The form has no authentication configured. Reviewer names are self-reported; they are not verified identities.

Restore HTML Report. restores report_html from the generated draft and retains the form fields. Build Approval Audit validates the decision, non-empty reviewer name, and parseable submission timestamp. It preserves the original fields and builds approval_audit.

Save approval  Audit inserts the audit details into the selected Data Table. Restore Review Context matches the saved row back to the original item, checks the audit event identifier, and restores the report and review fields. It adds the saved row identifier as audit_storage.

Review Decision Gate passes only an exact Approve decision to Update Approved Report. That node checks approval, reviewer name, HTML presence, and the expected pending markers. It replaces those markers with approval wording and escapes the reviewer name.

Create HTML Report File converts report_html to a downloadable file named SAP_RAID_Governance_Report.html. The export has an unusual binary output property value of `=`; file download worked during the session. Normalising this property can be considered separately if downstream integrations need a conventional binary field.

## Recreating the prototype elsewhere

1. Use an n8n installation supporting the node types in the export, including Data Table and the Groq language-model integration. The exact installed n8n version was not captured.
2. Import the supplied workflow JSON.
3. Create or select a Groq credential in Groq Chat Model. Credential references in workflow JSON do not replace configuring the actual credential.
4. Create a Data Table named SAP RAID approval audit in the intended n8n project. The workflow export references the original table; it does not contain the table's stored audit rows.
5. Add the eleven String columns listed below. Keep n8n's system columns.
6. In Save approval  Audit, select the newly created table, retain the Row / Insert action, and map each column from the matching property inside approval_audit.
7. Keep the connection order: Build Approval Audit → Save approval  Audit → Restore Review Context → Review Decision Gate.
8. Start a manual run and submit the generated review form. Verify the saved row and the chosen branch before relying on the imported workflow.

| String column | Value in Save approval  Audit |
|---|---|
| audit_event_id | `{{ $json.approval_audit.audit_event_id }}` |
| execution_id | `{{ $json.approval_audit.execution_id }}` |
| workflow_id | `{{ $json.approval_audit.workflow_id }}` |
| report_title | `{{ $json.approval_audit.report_title }}` |
| decision | `{{ $json.approval_audit.decision }}` |
| reviewer_name | `{{ $json.approval_audit.reviewer_name }}` |
| review_comments | `{{ $json.approval_audit.review_comments }}` |
| submitted_at | `{{ $json.approval_audit.submitted_at }}` |
| recorded_at | `{{ $json.approval_audit.recorded_at }}` |
| reviewer_identity_source | `{{ $json.approval_audit.reviewer_identity_source }}` |
| form_mode | `{{ $json.approval_audit.form_mode }}` |

## Running and reviewing

Start the manual workflow, wait for the review form, and enter a decision, reviewer name, and comment. Approve should save a row and produce an approved HTML file. Reject should save a row and take the false branch without generating an approved file. Review audit rows in the project's Data Tables view.

An execution of a single node can reuse upstream test data. Inspect the execution and submission timestamps when interpreting results. Re-executing the Insert node can create another row for the same audit event: duplicate prevention is not implemented.

## Backup and remaining work

Keep the workflow JSON, a separate export/backup of the audit table, and the n8n installation's normal data backup. The JSON alone is not a full runtime backup.

Next portfolio steps: capture a current workflow screenshot, preserve approval/rejection/invalid-count evidence, write the case study, and record a short demonstration. Do not claim time savings, client outcomes, live SAP integration, authenticated approval, immutable audit history, or production readiness without further evidence.

Further engineering tests can cover duplicate retries, unavailable storage, missing review fields, malformed AI output, HTML escaping, and other validation rules. An execution identifier links an audit entry to its run; the prototype does not yet store a hash or immutable version of the exact reviewed report.

No workflow files were changed as part of preparing these notes.
