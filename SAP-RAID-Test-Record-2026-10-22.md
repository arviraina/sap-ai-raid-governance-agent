# SAP AI RAID Governance Agent — test record

Date: 22 October 2026. Environment: local n8n; fictional training actions; manual test forms.

Evidence basis: live screen observations in this chat plus static review of the supplied workflow export. This is a record of observed manual tests, not an automated regression-suite result. Browser screenshots and table backups are not bundled with this document.

| Scenario | Expected behaviour | Observed result | Assessment |
|---|---|---|---|
| Initial audit insert | Store decision, reviewer, comments, timestamps and run identifiers; return saved row identifier | Approved test submission produced audit row 1; restore-context output retained the HTML and review details | Passed for the observed sample; reused earlier submission data |
| Fresh rejection | Store Reject and test comment; route to false branch; skip approved-report generation | Execution 110 produced audit row 2 with Reject, reviewer and rejection-test comment. False branch received one item; approved-report and file nodes did not run | Passed end to end for the observed run |
| Fresh approval | Store Approve and test comment; route to true branch; generate report file | Execution 111 produced audit row 3 with Approve, reviewer and approval-test comment. Approved-report and file nodes succeeded with one output item | Passed end to end for the observed run |
| Incorrect AI RED count | Detect mismatch and block the validation gate | Temporary input node changed the AI RED count to 999 while source count remained 2. Validator returned FAIL and Incorrect RED count; validation gate routed the item to false | Passed at validator and gate level; not a fresh full-workflow run |
| Restore normal input | Remove test node and recover normal validation | AI RAID Analysis was reconnected directly to Validate AI Output; validator passed and the gate took its true branch | Passed at validator and gate level |

The approved HTML report viewed during the session showed data validation PASSED, PM review APPROVED with the test reviewer name, and APPROVED FOR REVIEWED DISTRIBUTION. The fresh approval test subsequently confirmed successful file generation; that new file's full contents were not independently re-inspected.

## Export checks

The supplied export contains all three audit-related nodes, eleven mapped String columns, and the restore-context link before the review gate. It contains no Test Invalid AI Data node and no pinned data. AI RAID Analysis connects directly to Validate AI Output. It is marked inactive.

## Coverage limits

Only the incorrect RED-count case was tested for invalid AI data. Missing summaries, mismatched owners, duplicate actions and other validator checks were inspected in code but not exercised live in this session. Reviewer authentication, duplicate insert prevention, storage-failure recovery, concurrent reviews, exact-report version linkage and production deployment were not tested.

## Progress that can be recorded manually

- Record approval audit trail: prototype milestone complete for the tested behaviour.
- Test approval, rejection and invalid data: initial three-scenario check complete; broader coverage remains.
- Document setup and workflow: notes prepared alongside this record; a clean-environment import has not been tested.
- Case study and demonstration: next portfolio deliverables.

Career Compass is browser-local and was not automatically updated by these tests or this document.
