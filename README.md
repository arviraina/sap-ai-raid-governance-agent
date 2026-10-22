# SAP AI RAID Governance Agent

An n8n prototype for AI-assisted SAP action review, with source-based validation, human approval, persistent audit records and HTML reporting.

## What it does

- Filters fictional SAP project actions to identify overdue items.
- Applies rules-based RED, AMBER and GREEN classifications.
- Uses Groq to draft structured analysis, follow-up actions and evidence gaps.
- Validates selected AI output against source facts before generating a draft report.
- Collects a human approval or rejection and saves an audit record in n8n Data Tables.
- Generates an approved HTML report only on the approval path.

## Workflow

Sample actions → overdue filter → risk classification → AI analysis → validation → review form → audit storage → decision gate → approved HTML report.

Rejected reviews are recorded and stop before approved-report generation. Failed validation takes the false branch before the report and review stages.

## Project files

- [Portfolio case study](SAP-RAID-Portfolio-Case-Study.md)
- [Setup and workflow notes](SAP-RAID-Setup-and-Workflow-Notes.md)
- [Manual test record](SAP-RAID-Test-Record-2026-10-22.md)
- [Workflow JSON for import](SAP-RAID-Workflow-Shareable.json)

## Setup

Import the shareable JSON into n8n, configure your own Groq credential, and create/select the audit table using the columns and mappings in the setup notes.

The JSON contains no saved credential reference or configured destination table. Existing audit rows are not included. Import into a clean environment has not yet been tested.

## Tested behaviour

- Approval saved an audit row and generated a report file.
- Rejection saved a row and skipped the approved-report nodes.
- A deliberately incorrect RED count returned validation FAIL and went to the false branch.
- Normal validation passed again after removing the temporary test node.

## Scope and limitations

This is a local portfolio prototype using fictional training data. It has no live SAP integration or automated stakeholder distribution. Reviewer names are self-reported.

The audit table is editable; duplicate insert prevention, immutable report-version linkage and broader test coverage remain future improvements. No business savings or production-readiness claims have been measured.

## Tools

n8n · Groq · JavaScript · Structured Output Parser · n8n Data Tables · HTML
