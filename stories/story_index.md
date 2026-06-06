# Story Index

> **Next Story ID: US-005**
> *(Increment this after each new story is added)*

---

## How This Works

Paste your story into chat and run `/new-story` — everything below happens automatically:
1. Story file created as `stories/US-NNN_title.md`
2. Registered here with status 🔄 In Progress
3. Loaded into `current_story.md`
4. Review → Risk Analysis → Test Cases generated
5. Outputs saved to `stories/outputs/US-NNN_*.md`
6. Status updated to ✅ Complete

---

## Status Legend

| Status | Meaning |
|--------|---------|
| 🔲 Not Started | Story registered but `/new-story` not yet run |
| 🔄 In Progress | Currently being processed by the skills |
| ✅ Complete | All three outputs generated and saved |
| ⏸ On Hold | Blocked or deferred |

---

## Stories

| Story ID | Title | Sprint | Status | Story File | Outputs |
|----------|-------|--------|--------|------------|---------|
| US-001 | Customer Registration | Sprint 1 | 🔲 Not Started | [US-001_customer_registration.md](US-001_customer_registration.md) | — |
| US-002 | Submit a Claim | Sprint 1 | 🔲 Not Started | [US-002_submit_claim.md](US-002_submit_claim.md) | — |
| US-003 | Password Reset | Sprint 1 | ✅ Complete | [US-003_password_reset.md](US-003_password_reset.md) | [Tests](outputs/US-003_test_cases.md) · [Risk](outputs/US-003_risk_analysis.md) · [Summary](outputs/US-003_execution_summary.md) |
| US-004 | Submit Insurance Claim | Sprint 2 | ✅ Complete | [US-004_submit_insurance_claim.md](US-004_submit_insurance_claim.md) | [Tests](outputs/US-004_test_cases.md) · [Risk](outputs/US-004_risk_analysis.md) · [Summary](outputs/US-004_execution_summary.md) |

---

## Output Files Per Story

| File | Saved As |
|------|---------|
| `outputs/generated_test_cases.md` | `stories/outputs/US-NNN_test_cases.md` |
| `outputs/risk_analysis.md` | `stories/outputs/US-NNN_risk_analysis.md` |
| `outputs/execution_summary.md` | `stories/outputs/US-NNN_execution_summary.md` |
