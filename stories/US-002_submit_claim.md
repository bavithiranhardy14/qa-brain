# US-002 — Submit a Claim

## Story Details

**Story ID:** US-002
**Title:** Submit a Claim
**Epic:** Claims Management
**Sprint:** Sprint 1
**Status:** 🔲 Not Started

---

## User Story

> As a **customer with an active health insurance policy**,
> I want to **submit a medical claim through the portal**,
> So that **I can be reimbursed for eligible medical expenses**.

---

## Acceptance Criteria

- [ ] AC-1: Customer can only submit a claim against an active policy
- [ ] AC-2: Customer fills in claim form with date of treatment, hospital name, diagnosis, and claim amount
- [ ] AC-3: Customer uploads at least one supporting document (bill, prescription, or discharge summary)
- [ ] AC-4: System validates the claim amount does not exceed the remaining Sum Insured
- [ ] AC-5: System rejects the claim if the date of treatment falls within the waiting period
- [ ] AC-6: System rejects the claim if the date of treatment is more than 90 days in the past
- [ ] AC-7: On successful submission, customer receives a Claim Reference Number and email confirmation
- [ ] AC-8: Submitted claim appears in the customer's claim history with status "Pending Review"

---

## Assumptions

- Deductible and co-payment calculations happen at the Claims Handler review stage, not at submission
- Only reimbursement claims are in scope (cashless claims are out of scope for Sprint 1)
- Customer must have at least one active policy to see the "Submit Claim" option

## Out of Scope

- Cashless / pre-authorization claims — Sprint 2
- Claim appeal workflow — future sprint

## Dependencies

- Policy Management module must be live (to validate active policies and remaining Sum Insured)
- Document upload service must be available
- Email notification service must be configured

## Notes / Additional Context

- Allowed document formats: PDF, JPG, PNG — max 5 MB per file, max 10 files per claim
- Claim submission window: within 90 days of date of treatment
- Waiting period: 30 days from policy start for illness; 90 days for pre-existing conditions
