# Product Support Engineer Assignment – Submission

- **Name:** Anup Prakash Naik
- **Date submitted:** 14 November 2026
- **Repository link:** https://github.com/anupnaik01/complyance-product-support-assignment
- **Total time spent (real time):** Approximately 8 hours

> All customer communications below are proposed drafts written against the exercise clock. They were not sent.

- **Items fully investigated, and why I chose them:**
  - Al Noor Fuel: Reproducible BR-CO-15 calculation rejection.
  - Sahara Medical: Reproducible HTTP 429 submission handling defect.
  - Ghaf Tree Café: Reproducible date-format validation issue.
  - Oasis Builders and API restart alert: Investigated, but historical root causes remain unconfirmed.

---

## Incident 1 — Oasis Builders

### Acknowledgement

- **Proposed time:** 09:15 Dubai time (draft only; not sent).

> Hello Rashid,
>
> Thank you for reporting the upload issue. I understand that your November workbook appears stuck and that you are seeing duplicate document number messages.
>
> We will check the invoice-level processing history and ASP references to establish which invoices have already been received.
>
> Please avoid uploading the entire workbook again until we reconcile the previous attempts.
>
> I will share an update by 11:15 Gulf time.
>
> Regards,
> Product Support Team

### Priority and justification

- **Priority:** P2 – provisional.
- **Why:** The customer reports a stuck monthly upload and duplicate-number messages. Actual delivery impact from the original attempts could not be established. Upgrade to P1 if no invoices are reaching the ASP.
- **SLA:** First response deadline 10:05 Dubai time; proposed next update 11:15; resolution target 8 hours.

### Investigation log

| Exercise time | Check | Finding |
|---|---|---|
| Not recorded | Read incident and identified workbook. | Three upload attempts and duplicate messages reported. |
| Not recorded | Ran `make replay CUSTOMER=oasis`. | Controlled replay completed with eight invoices per import. |
| Not recorded | Queried original invoice numbers. | Original invoice numbers were not found under the queried seller. |
| Not recorded | Queried replay imports. | All 16 replay invoices were Accepted with ASP references. |
| Not recorded | Inspected duplicate validation code. | Duplicate check compares seller, invoice number and other imports, excluding rejected records. |

### Dead ends and next checks

- The controlled replay completed successfully, so it did not reproduce the original stuck-upload behaviour.
- The original invoice numbers were not found in the seller records queried. This does not prove that the original uploads were never received.
- Inspecting duplicate-validation code confirmed that duplicate checks exist, but did not establish what happened during the customer's historical upload attempts.
- **Next checks:** Obtain the original import IDs and timestamps, inspect the corresponding historical API and import logs, and reconcile each invoice against its ASP message reference before recommending resubmission.

### Root cause

- **What happened:** Original customer-reported duplicate messages could not be tied conclusively to a historical import. Controlled replay did not reproduce the stuck-upload behaviour.
- **Where:** `services/api/internal/store/store.go:296–306`; `services/api/internal/validate/validate.go:73–81`; `services/api/internal/importer/importer.go:100–118`.
- **Why:** Duplicate detection exists, but original upload outcome and ASP delivery status remain unconfirmed.
- **Evidence:** `evidence/oasis-replay.log`, `evidence/oasis-code-findings.txt`, `evidence/oasis-directory-metrics.txt`.
- **Confidence and open questions:** Duplicate validation is confirmed in code. Historical root cause and original invoice delivery status remain unknown. Original import IDs, historical logs and ASP references need investigation.

### Customer resolution reply OR escalation note

- **Which:** Engineering/support escalation because original upload state and ASP delivery status remain unconfirmed.

> Priority: P2 provisional.
> Customer: Oasis Builders Supply LLC.
> Reporter: Rashid Al Mansoori.
> Symptoms: Processing appears stuck; duplicate document number messages after multiple attempts.
> Evidence: Controlled replay completed successfully. Original invoice numbers were not found in queried local records.
> Tried: Replayed sample, queried imports and inspected duplicate validation code.
> Not confirmed: Original import outcome and ASP acceptance status.
> Request: Identify original import attempts and reconcile invoice-level ASP delivery status.
> Workaround: Hold full-batch resubmission until reconciliation is complete.
> Proposed next customer update: 11:15 Dubai time.

### Workaround offered

Do not upload the entire workbook again. Reconcile each original invoice against its import history and ASP reference. Resubmit only invoices confirmed as not received.

### Prevention ideas

- Provide invoice-level status and ASP reference visibility.
- Make retry behaviour idempotent.
- Improve duplicate-number error messages.
- Alert support when an import remains in Processing unusually long.
- Retain searchable import-attempt history.

---

## Incident 2 — API Restart Alert / Marina Bay

### Acknowledgement

- **Proposed time:** 09:50 Dubai time (draft only; not sent).
- **Contact:** Priya Nair.

> Hello Priya,
>
> Thank you for forwarding the alert. We understand the reported API restarts, intermittent 502 responses and invoices remaining in Validated status.
>
> We will investigate API health, container restart history, resource usage, request metrics and the submission queue. We will also check whether the reported behaviour can be reproduced under controlled load.
>
> We will provide another update by 11:50 Gulf time.
>
> Regards,
> Product Support Team

### Priority and justification

- **Priority:** P2 – provisional.
- **Why:** Multiple customers reportedly experienced intermittent 502 errors and delayed submissions. A complete production outage was not established. Upgrade to P1 if no invoices can reach the ASP.
- **SLA:** First response deadline 10:42 Dubai time, calculated from the original 09:42 alert; proposed next update 11:50; resolution target 8 hours.

### Investigation log

| Exercise time | Check | Finding |
|---|---|---|
| Not recorded | Inspected API container restart and OOM information. | Local API had RestartCount 0 and OOMKilled false. |
| Not recorded | Checked API container configuration. | Memory limit was 128 MiB; restart policy was unless-stopped. |
| Not recorded | Reviewed baseline resource usage. | Memory was approximately 33.59 MiB of 128 MiB. |
| Not recorded | Reviewed API logs and metrics. | Observed successful requests; historical evidence of the reported restart sequence was not established. |
| Not recorded | Ran controlled peak test with 20 uploads and concurrency 2. | 20 imports completed successfully, with 120 invoices accepted. |
| Not recorded | Checked resource usage after the test. | Memory increased to approximately 61.95 MiB; RestartCount remained 0 and OOMKilled remained false. |

### Root cause

- **What happened:** The reported historical API restarts were not reproduced in the local environment. A controlled test completed successfully without a restart.
- **Where:** API configuration in `docker-compose.yml`; runtime configuration in `services/api/Dockerfile`.
- **Why:** Historical cause remains unconfirmed. The 128 MiB memory limit is worth monitoring, but the controlled test did not demonstrate an out-of-memory failure.
- **Evidence:** `evidence/api-compose-config.txt`, `evidence/api-dockerfile.txt`, `evidence/api-inspect-before.txt`, `evidence/api-stats-before.txt`, `evidence/api-controlled-load-test.log`, `evidence/api-test-after.txt`, `evidence/api-test-stats-after.txt`, `evidence/api-metrics-before.txt`, `evidence/api-metrics-after.txt`, `evidence/incident-02-test-findings.txt`, `evidence/grafana-before.png`.
- **Confidence and open questions:** The controlled test did not reproduce a restart. The historical cause remains unknown. Historical Grafana alert data, container events, host resource metrics and API logs for the reported 15-minute period are still needed.

### Dead ends and next checks

- The local API container showed RestartCount 0 and OOMKilled false. This did not confirm the reported historical restart.
- The controlled test completed 20 imports successfully, with 120 invoices accepted and no container restart.
- The 128 MiB memory limit is worth monitoring, but the test did not establish an out-of-memory issue.
- **Next checks:** Review historical Grafana metrics around the 09:42 alert, Docker container events, host memory and CPU metrics, and API logs for the reported 502 responses. Correlate the findings with invoices that remained in Validated status.

### Customer resolution reply OR escalation note

- **Which:** Engineering escalation because the reported historical restarts and intermittent 502 responses require production telemetry review.

> Priority: P2 provisional.
> Contact: Priya Nair / Marina Bay go-live readiness.
> Reported: API restarted three times in 15 minutes; intermittent 502 responses; invoices reportedly remained Validated since 09:30.
> Evidence: Local API had no restart and no OOM kill. Controlled load test completed 20 uploads with 120 accepted invoices. Memory increased from approximately 33.59 MiB to 61.95 MiB.
> Tried: Reviewed container configuration, inspect output, logs, metrics and controlled load-test results.
> Not confirmed: Historical production root cause.
> Request: Review historical container events, Grafana metrics, host resource usage and API 502 logs.
> Proposed next customer update: 11:50 Dubai time.

### Workaround offered

Monitor API health and invoice states. Reconcile invoices remaining Validated before retrying. Do not blindly resubmit invoices that may already have reached the ASP.

### Prevention ideas

- Alert on restart count, OOM kills, elevated memory and HTTP 5xx responses.
- Monitor the age of invoices remaining Validated.
- Retain historical container events and request correlation IDs.
- Run load tests at expected go-live concurrency.
- Review API memory limits against measured peak usage.

---

## Incident 3 — Al Noor Fuel: BR-CO-15 Rejections

### Acknowledgement

- **Customer:** Fatima Khoury, Al Noor Fuel & Lubricants LLC.
- **Proposed time:** 10:30 Dubai time (draft only; not sent).
- **Priority:** P2.
- **First response deadline:** 11:15 Dubai time.
- **Proposed next update:** 12:30 Dubai time.

> Hello Fatima,
>
> Thank you for sharing the validation errors. We understand that some November invoices were accepted while others were rejected with BR-CO-15.
>
> We are checking the invoice totals and tax calculations against the reported rule. Please retain the accepted invoice references and avoid uploading the entire workbook again while we reconcile the rejected invoices.
>
> We will share our findings and next steps by 12:30 Gulf time.
>
> Regards,
> Product Support Team

### Priority and justification

P2 because a subset of invoices was rejected by the ASP, while other invoices were accepted. This affects invoice processing and requires correction, but the evidence does not show a complete service outage.

### Investigation log

| Exercise time | Check | Finding |
|---|---|---|
| Not recorded | Replayed Al Noor scenario. | Import `e8276c04-84d5-4b8d-9449-d6311d449212` contained 6 invoices: 3 Accepted and 3 Rejected. |
| Not recorded | Queried invoice statuses and ASP references. | Invoices 0041, 0043 and 0045 were Accepted with ASP references. |
| Not recorded | Inspected rejected invoice errors. | Invoices 0042, 0044 and 0046 were rejected by the ASP with BR-CO-15. |
| Not recorded | Recalculated net, VAT and gross amounts. | Differences were 0.01, 0.02 and 0.01 respectively. |
| Not recorded | Inspected API PINT mapping and validation code. | Net, VAT and gross totals are calculated and rounded through separate operations. |
| Not recorded | Inspected ASP validation code. | ASP checks that tax-inclusive amount equals tax-exclusive amount plus VAT. |

### Root cause

- **Root cause:** The API calculates and rounds net, VAT and gross totals separately. The gross calculation can differ from the sum of the rounded net and VAT amounts.
- **API code:** `services/api/internal/pint/pint.go`, lines 135–152 and 156–186.
- **Validation code:** `services/api/internal/validate/validate.go`, lines 150–158.
- **ASP validation:** `services/asp-mock/main.go`, lines 225–229.
- **Reproduction:** The three rejected invoices showed these discrepancies:
  - Invoice 0042: got 15022.42; expected 15022.43.
  - Invoice 0044: got 11817.57; expected 11817.59.
  - Invoice 0046: got 5610.99; expected 5611.00.
- **Evidence:** `evidence/al-noor-replay.log`, `evidence/al-noor-db-findings.txt`, `evidence/al-noor-code-findings.txt`.
- **Confidence:** High. The controlled replay, database results and source code support the same explanation.

### Dead ends and checks performed

- The three accepted invoices were not part of the rejection issue; they had valid ASP references.
- The investigation did not identify an issue affecting all six invoices or a complete ASP outage.
- The rejected invoices showed small monetary differences, so the investigation focused on net, VAT and gross calculations rather than assuming incorrect source invoice data.
- **Further check:** After a rounding correction is implemented, rerun the three rejected invoices and verify that the final gross amount equals the rounded net amount plus rounded VAT before resubmission.

### Customer resolution reply OR escalation note

- **Which:** Engineering escalation for correction of monetary rounding and regression testing.

> Priority: P2.
> Customer: Al Noor Fuel & Lubricants LLC.
> Contact: Fatima Khoury.
> Reproduction: Controlled replay produced 3 accepted and 3 rejected invoices.
> Evidence: Rejected invoices 0042, 0044 and 0046 failed BR-CO-15. Their gross amounts differed from net plus VAT by 0.01, 0.02 and 0.01.
> Suspected component: PINT amount calculation in `services/api/internal/pint/pint.go`, lines 135–186.
> ASP validation: `services/asp-mock/main.go`, lines 225–229.
> Request: Review the rounding strategy and add regression tests to ensure the submitted gross amount equals the rounded net amount plus rounded VAT amount.
> Workaround: Retain accepted references and correct only confirmed rejected invoices after the fix and safe resubmission process are verified.
> Proposed next customer update: 12:30 Dubai time.

### Workaround offered

Keep the three accepted ASP references. Do not resend the complete workbook. Correct and resubmit only the three rejected invoices after engineering verifies the calculation fix and the safe resubmission procedure.

### Prevention ideas

- Use a consistent monetary rounding strategy across net, VAT and gross calculations.
- Add regression tests for one-cent and multi-cent rounding differences.
- Validate the final gross amount against net plus VAT before ASP submission.
- Preserve invoice-level rejection reasons and accepted ASP references.
- Provide a clear customer explanation of which invoices need correction.

---

## Incident 4 — Bluepeak / Sahara Medical: Invoices Not Received by ASP

### Acknowledgement

- **Customer contact:** Daniel Okafor.
- **Proposed time:** 11:40 Dubai time (draft only; not sent).
- **Priority:** P2.
- **First response deadline:** 12:30 Dubai time.
- **Proposed next update:** 13:40 Dubai time.

> Hello Daniel,
>
> Thank you for reporting the missing ASP references. We understand that your 40-invoice import was accepted by the portal, but several invoices do not yet show ASP references.
>
> We are checking the invoice-level submission status and ASP responses to identify which invoices were received. Please do not resend the full batch until we complete reconciliation.
>
> We will provide another update by 13:40 Gulf time.
>
> Regards,
> Product Support Team

### Priority and justification

P2 because only part of the batch reached the ASP, leaving 32 invoices without ASP references in the controlled replay. This is a significant processing issue, but the replay does not establish a complete production outage.

### Investigation log

| Exercise time | Check | Finding |
|---|---|---|
| Not recorded | Replayed Sahara scenario. | Import `033d0614-0cfa-4e9b-a9c6-9ae4c2f10025` contained 40 invoices. |
| Not recorded | Queried invoice statuses and ASP references. | 8 invoices were Accepted with ASP message IDs; 32 remained Submitted without ASP references. |
| Not recorded | Reviewed API and ASP service logs. | The first 8 submissions were accepted. Subsequent requests received HTTP 429 rate-limit responses. |
| Not recorded | Checked rate-limit log details. | ASP responses included `retry_after_s:1`. |
| Not recorded | Inspected submission handling code. | The code handles HTTP 200, 422 and selected server errors, but has no dedicated HTTP 429 retry handling. |

### Root cause

- **Root cause:** The API does not handle HTTP 429 rate-limit responses with a retry process. In the submission flow, invoices are marked Submitted before the ASP request. When the ASP returns 429, the code does not move those invoices into a retryable state or process the `Retry-After` value. They remain Submitted without an ASP reference.
- **Code location:** `services/api/internal/submit/submit.go`, especially lines 114–117 and 133–155.
- **Reproduction:** The controlled replay accepted 8 invoices and left 32 Submitted without ASP message IDs after rate-limit responses.
- **Evidence:** `evidence/sahara-replay.log`, `evidence/sahara-db-findings.txt`, `evidence/sahara-service-logs.txt`, `evidence/sahara-code-findings.txt`.
- **Confidence:** High. The replay, database states, service logs and source code consistently support this finding.

### Dead ends and checks performed

- The eight invoices with ASP message IDs were already accepted and should not be resubmitted.
- The remaining 32 invoices had no ASP references, so their final delivery status could not be assumed from the portal status alone.
- The ASP returned HTTP 429 with a retry delay, but the current submission code did not implement the required retry handling.
- **Further check:** Reconcile the 32 invoices with ASP records and verify an idempotent retry process that respects `Retry-After` before attempting resubmission.

### Customer resolution reply OR escalation note

- **Which:** Engineering escalation for missing rate-limit retry handling.

> Priority: P2.
> Customer: Bluepeak / Sahara Medical.
> Contact: Daniel Okafor.
> Reproduction: 40-invoice controlled replay; 8 Accepted and 32 remained Submitted without ASP references.
> Evidence: ASP logs show HTTP 429 responses with `retry_after_s:1`.
> Suspected component: `services/api/internal/submit/submit.go`, lines 114–117 and 133–155.
> Request: Implement HTTP 429 handling, respect Retry-After, use bounded backoff and safe idempotent retries, and expose a clear retry-pending state.
> Workaround: Retain the 8 accepted ASP references. Reconcile the remaining invoices and retry only those confirmed as not received through a controlled process.
> Proposed next customer update: 13:40 Dubai time.

### Workaround offered

Keep the 8 accepted ASP references. Reconcile the remaining 32 invoices against ASP records before retrying. Retry only invoices confirmed as not received, respecting the ASP retry delay and using an idempotent process. Do not resend all 40 invoices.

### Prevention ideas

- Add explicit HTTP 429 handling.
- Read and respect the ASP `Retry-After` response.
- Implement bounded exponential backoff and retry limits.
- Use idempotency to prevent duplicate invoice submissions.
- Track retry-pending and retry-exhausted states separately.
- Alert on invoices that remain Submitted without an ASP reference beyond a defined time.

---

## Incident 5 — Ghaf Tree Café: Date Format Validation Error

### Acknowledgement

- **Customer:** Hessa Al Ketbi, Ghaf Tree Café, Fujairah.
- **Proposed time:** 12:20 Dubai time (draft only; not sent).
- **Priority:** P3.
- **First response deadline:** 16:10 Dubai time.
- **Proposed next update:** 16:00 Dubai time.

> Hello Hessa,
>
> Thank you for sharing the validation error. We understand that your invoices were rejected because the date values were provided in DD/MM/YYYY format.
>
> Our initial investigation indicates that the current import accepts ISO date format (YYYY-MM-DD) or valid Excel date cells. Please try converting the affected issue dates and due dates to the accepted format before uploading again.
>
> We will share further guidance by 16:00 Gulf time.
>
> Regards,
> Product Support Team

### Priority and justification

P3 because the issue affects one customer's workbook format and has a practical workaround. The evidence does not indicate a platform-wide failure.

### Investigation log

| Exercise time | Check | Finding |
|---|---|---|
| Not recorded | Replayed Ghaf scenario. | Import `983c351b-bed8-4c58-85b6-f9a69c2833fa` contained 4 invoices. |
| Not recorded | Checked import and invoice statuses. | All 4 invoices were rejected. |
| Not recorded | Reviewed validation errors. | There were 16 date validation errors across the invoices, affecting issue date and due date fields. |
| Not recorded | Inspected date validation code. | `validate.go` calls `sheet.ParseDate` for both issue and due dates. |
| Not recorded | Inspected date parsing implementation. | `ParseDate` accepts ISO `YYYY-MM-DD` and Excel date serial values, but not slash-formatted date text such as `13/11/2026`. |

### Root cause

- **Root cause:** The uploaded workbook contains dates as DD/MM/YYYY text. The current parser does not support this text format, even when the date itself is valid.
- **Validation code:** `services/api/internal/validate/validate.go`, lines 113–117.
- **Date parser:** `services/api/internal/sheet/sheet.go`, lines 167–185.
- **Reproduction:** All four invoices in the controlled replay were rejected. The database recorded 16 date validation errors.
- **Evidence:** `evidence/ghaf-replay.log`, `evidence/ghaf-code-findings.txt`.
- **Confidence:** High. The replay results and parser implementation confirm the format limitation.

### Dead ends and checks performed

- All four invoices were rejected with date validation errors; the investigation did not indicate a platform-wide import failure.
- The date values were valid-looking dates, but the parser did not accept DD/MM/YYYY text format.
- The issue was traced to date parsing rather than an incorrect invoice amount or VAT calculation.
- **Further check:** Test a corrected workbook using ISO date text and valid Excel date cells, then verify that the invoices pass date validation.

### Customer resolution reply OR escalation note

- **Which:** Customer resolution guidance and product feedback for broader date-format support.

> Priority: P3.
> Customer: Ghaf Tree Café.
> Contact: Hessa Al Ketbi.
> Reproduction: Controlled replay of 4 invoices resulted in 4 rejected invoices and 16 date validation errors.
> Error: AE-BR-04 on issue date and due date fields.
> Cause: The parser accepts ISO dates and Excel date serials but does not accept DD/MM/YYYY text.
> Code: `services/api/internal/sheet/sheet.go`, lines 167–185.
> Workaround: Convert date text such as `13/11/2026` to `2026-11-13`, or use valid Excel date cells.
> Proposed next customer update: 16:00 Dubai time.

### Workaround offered

Convert both issue dates and due dates from DD/MM/YYYY text to ISO YYYY-MM-DD text. For example, convert `13/11/2026` to `2026-11-13`. Alternatively, use valid Excel date cells. Check the corrected workbook before uploading again.

### Prevention ideas

- Consider supporting common date formats such as DD/MM/YYYY.
- Provide a clear validation message with an example of the accepted date format.
- Identify the exact worksheet row and date column containing invalid values.
- Add parser tests for ISO dates, Excel serial dates and common regional date formats.
- Document supported workbook formats for customers.

---

# Overall Assessment

## 1. Incident prioritisation

My initial prioritisation was based on customer impact, number of affected invoices, business urgency and the possibility of invoices being lost or duplicated.

| Incident | Priority | Reason |
|---|---|---|
| Oasis Builders | P2 provisional | Month-end upload stuck, repeated attempts and possible duplicate submissions. |
| API Restart Alert | P2 provisional | Multiple customers reported intermittent 502 responses and invoices remaining Validated. |
| Al Noor Fuel | P2 | A subset of invoices was rejected by the ASP due to amount calculation differences. |
| Sahara Medical / Bluepeak | P2 | Only 8 of 40 invoices were accepted in the controlled replay; 32 remained without ASP references after HTTP 429 responses. |
| Ghaf Tree Café | P3 | One customer's date format caused validation failures, with a practical formatting workaround. |

The priorities are provisional where historical production impact could not be confirmed. I would upgrade an incident to P1 if evidence showed a complete production outage or that no invoices were reaching the ASP.

As investigation progressed, the Sahara issue became a particularly important engineering concern because the replay and code inspection demonstrated missing HTTP 429 retry handling.

## 2. What I would do differently next time

- Capture original import IDs, invoice statuses and ASP references before running replays.
- Preserve historical logs, Grafana data and container events before conducting tests.
- Keep controlled replay results separate from original customer records.
- Reconcile invoice-level statuses before recommending any resubmission.
- Record investigation timestamps and actions as I perform them.
- Start with the smallest useful reproduction and then expand testing.
- Escalate confirmed code issues with reproduction steps, evidence and exact file locations.
- Communicate confirmed facts separately from assumptions and unverified historical causes.

## 3. Key learning from this assignment

- Learned how the portal, API, PostgreSQL database and ASP mock interact in an e-invoicing workflow.
- Understood that HTTP 201 from the import endpoint does not mean every invoice has been accepted by the ASP.
- Practised checking invoice-level statuses and ASP message references.
- Investigated monetary rounding differences causing BR-CO-15 failures.
- Learned why HTTP 429 responses need Retry-After handling and safe retries.
- Investigated date parsing and supported Excel date formats.
- Practised using Docker Compose, PostgreSQL queries, Grafana, Prometheus and Loki for troubleshooting.
- Improved my ability to connect customer symptoms with logs, database records and source code.

## 4. What I want to learn next

- Peppol network architecture and PINT AE requirements in greater depth.
- UAE e-invoicing validation and compliance rules.
- Distributed tracing and correlation IDs.
- Queue management, retry strategies and idempotent API design.
- Production incident management, monitoring and alert configuration.
- Financial precision and decimal rounding in invoice processing.

## Portal and Grafana Evidence Review

### Portal – Batch Overview

Reviewed the Invoicely Lite Batches page after running the controlled incident replays.

The portal displayed the following batch-level results:

- Al Noor Fuel: 3 accepted and 3 rejected invoices.
- Sahara Medical: 8 accepted and 32 submitted invoices.
- Ghaf Tree Café: 4 rejected invoices.
- Oasis Builders: Two displayed batches, each showing 8 accepted invoices.

These portal observations are consistent with the controlled replay findings documented in the individual incident investigations. The Oasis batch results do not establish the outcome of the customer's original historical attempts.

**Evidence:** `evidence/portal-batches-overview.png`

### Grafana – Invoicely Lite Overview

Reviewed the Invoicely Lite – Overview dashboard using the Last 30 minutes time range.

At the time of the screenshot, the dashboard showed:

- API up: 1.
- API process: 1.
- Request rate: 0.327 requests per second.
- HTTP 5xx responses: 0.
- Submission queue: 0.
- Invoices accepted panel: 0.

The dashboard also displayed request-rate, latency and error-rate graphs.

These are snapshot observations from the displayed dashboard time window. They do not establish the historical production state during the earlier incidents or replace the controlled replay evidence.

**Evidence:** `evidence/grafana-overview-after-investigations.png`

## 5. Retrospective work log

| Date | From–to | What I worked on |
|---|---|---|
| 14 November 2026 | Exact start and end times not recorded; approximately 8 hours of work in total. | Set up the lab environment, uploaded and tested the sample workbook, investigated the five incidents using portal data, logs, database queries and source code, collected evidence, prepared customer communication drafts, and documented findings and recommendations. |

**Total working time:** Approximately 8 hours (retrospective estimate; breaks and non-working time excluded).

## 6. AI usage disclosure

I used ChatGPT as an assistance tool during this assignment to understand the instructions, plan investigation steps, explain terminal commands and SQL queries, interpret selected logs and source-code findings, and prepare documentation and customer communication drafts.

I verified the technical findings against the local lab using actual commands, replay results, database queries, logs and source-code inspection. I have identified historical causes as unconfirmed wherever the available evidence did not establish them.

All customer acknowledgements and replies in this submission are drafts for the exercise. They were not actually sent to customers.

---

# Final Submission Checklist

- [x] Confirm all five incident sections are present.
- [x] Check that the evidence files referenced above exist in the evidence directory.
- [x] Document the working date and estimated duration in the retrospective work log.
- [x] Update total time spent at the top of this document.
- [x] Review the acknowledgement and escalation drafts.
- [x] Check that no unverified historical cause is presented as confirmed.
- [x] Review the README and submission instructions.
- [x] Commit and push the completed assignment to my own GitHub repository.
- [ ] Submit the assignment form using the same email address used for the application.

