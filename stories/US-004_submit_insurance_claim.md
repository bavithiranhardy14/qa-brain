# US-004 — Submit Insurance Claim

## Story Details
**Story ID:** US-004
**Title:** Submit Insurance Claim
**Epic:** Claims Management
**Sprint:** Sprint 2
**Status:** 🔄 In Progress

## User Story
> As a policyholder, I want to submit a medical insurance claim online,
> so that I can get reimbursed for my medical expenses without visiting a branch.

## Acceptance Criteria
- [ ] AC-1: Policyholder can upload medical bills, prescriptions, and discharge summary (PDF/JPG, max 10MB each)
- [ ] AC-2: System validates that the claim amount does not exceed the remaining annual coverage limit
- [ ] AC-3: Claim is auto-approved if amount is below ₹5,000 and documents are complete
- [ ] AC-4: Claims above ₹5,000 are routed to a human reviewer within 24 hours
- [ ] AC-5: Policyholder receives email and SMS notification when claim status changes
- [ ] AC-6: Duplicate claim for the same hospital bill is rejected

## Assumptions
- Policyholder is authenticated and has at least one active policy
- The system has access to coverage limit data in real time
- "Documents complete" means all three document types (bills, prescription, discharge summary) are uploaded and pass format/size validation
- 24-hour routing SLA is based on calendar hours

## Out of Scope
- Claims for non-medical expenses
- Cashless claim processing (direct hospital billing)
- Claim appeals and re-submission workflow

## Dependencies
- Document Management module (upload, storage, validation)
- Notifications module (email + SMS)
- Policy Management module (coverage limit lookup)
- Claims Handler dashboard (manual review routing)
- Payment module (settlement trigger for auto-approved claims)

## Notes
- Waiting period of 30 days applies for new policies
- Pre-existing conditions have a 2-year exclusion period (⚠️ conflicts with BR-10 which states 90 days — needs clarification)
- Agent can submit claims on behalf of a policyholder

---

## QA Pipeline Results

**Pipeline run:** 2026-06-06 | **Status:** ✅ Complete

### 🔍 Review Story

| Check | Result |
|-------|--------|
| ACs clear? | ⚠️ Partial — AC-1 file size/format conflicts with business rules; AC-3 "documents complete" undefined |
| Ambiguities | 7 found — file size conflict, PNG omission, "documents complete" undefined, 24hr SLA scope, pre-existing exclusion contradiction, duplicate definition absent, SMS mandatory vs. optional |
| Missing ACs | 6 identified — waiting period, pre-existing exclusion, lapsed policy, 90-day window, agent submission constraints, claim status tracking |
| Testability | ⚠️ Issues — AC-3 auto-approval trigger not measurable; AC-4 SLA not auditable without timestamps; AC-6 duplicate logic undefined |

📄 [View Execution Summary](outputs/US-004_execution_summary.md)

---

### ⚠️ Analyze Risk

| Dimension | Score |
|-----------|-------|
| Business Logic | 9 / 10 |
| Security | 8 / 10 |
| Integration | 8 / 10 |
| Data Integrity | 8 / 10 |
| User Experience | 6 / 10 |
| Performance | 6 / 10 |
| **Overall** | **8 / 10 — 🔴 High** |

**Top risks:** Auto-approval threshold logic, file upload attack surface, coverage limit race conditions, pre-existing condition rule conflict, agent impersonation, notification reliability

📄 [View Risk Analysis](outputs/US-004_risk_analysis.md)

---

### ✅ Generate Test Cases

| Category | Count |
|----------|-------|
| Functional | 6 |
| Negative | 8 |
| Boundary | 7 |
| Integration | 5 |
| **Total** | **26** |

📄 [View Test Cases](outputs/US-004_test_cases.md)
