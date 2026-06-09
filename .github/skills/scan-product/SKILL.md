---
name: scan-product
description: 'Generate a full product test suite, module risk analysis, and missing integration report using only the knowledge base files — no user story required. Use when: scanning the full product for test coverage, generating a baseline test suite, identifying integration gaps across modules.'
argument-hint: 'Optionally specify a module name to scope the scan, or leave blank to scan the full product'
---

# Scan Product — Full Knowledge-Based QA Scan

## When to Use
- You want a full product test suite without writing a user story
- You want to see which modules carry the most risk before sprints begin
- You want to find integration gaps between modules
- No story is available yet, but QA prep needs to start

## What This Skill Does Automatically
1. Reads `knowledge/product_context.md` — modules, workflows, tech stack
2. Reads `knowledge/business_rules.md` — BR rules, validation rules, constraints, edge cases
3. Generates **Module Risk Map** — risk score for each module
4. Generates **Full Product Test Suite** — test cases derived from all workflows and business rules found in the knowledge files
5. Generates **Missing Integration Report** — cross-module data flows with no test coverage
6. Saves all three outputs to `outputs/product_scan/`
7. Generates a self-contained HTML report at `outputs/product_scan/product_qa_report.html`

---

## Procedure

### Step 1: Read Knowledge Files
Read both files in full:
- [knowledge/product_context.md](../../knowledge/product_context.md)
- [knowledge/business_rules.md](../../knowledge/business_rules.md)

Extract and hold in context:
- All module names and their descriptions
- All user workflows defined in the file
- All business rules defined in the file (however many exist)
- All validation rules with error messages
- All system constraints
- All edge case notes

---

### Step 2: Generate — Module Risk Map

For each module in the product, score risk across 4 dimensions (1–10):
- **Security Risk** — Does this module handle PII, payments, auth tokens, document uploads?
- **Business Logic Risk** — Does this module enforce financial rules, eligibility, approvals?
- **Integration Risk** — How many other modules does this module depend on?
- **Data Integrity Risk** — Does this module write/update critical records?

Calculate **Overall Module Risk** = average of 4 dimensions, rounded to 1 decimal.

Rank all modules from highest to lowest risk.

Flag modules with Overall Risk ≥ 7 as 🔴 High, 5–6.9 as 🟡 Medium, below 5 as 🟢 Low.

Write to `outputs/product_scan/module_risk_map.md` using this structure:

```markdown
# Product Module Risk Map
**Generated:** [date] | **Source:** knowledge base only (no story required)

## Risk Summary

| Module | Security | Business Logic | Integration | Data Integrity | Overall | Level |
|--------|----------|----------------|-------------|----------------|---------|-------|
| Claims Management | 9 | 9 | 8 | 9 | 8.8 | 🔴 High |
| ... | | | | | | |

## High Risk Modules (≥ 7.0)
[Detail each high-risk module with reason and recommended focus areas]

## Module Risk Notes
[Key observations — which modules need most test investment]
```

---

### Step 3: Generate — Full Product Test Suite

Generate test cases for **each workflow** defined in product_context.md.

For each workflow, generate:
1. **Functional Tests** — happy path for each major step in the workflow
2. **Negative Tests** — invalid inputs, unauthorized access, failed dependencies
3. **Boundary Tests** — min/max values from business_rules.md (coverage limits, file sizes, timeouts, password rules, etc.)
4. **Business Rule Tests** — one test per relevant business rule that applies to this workflow

**TC-ID format:** `TC-[MODULE_CODE]-NNN`
- Example: `TC-CLM-001` for Claims, `TC-POL-001` for Policy, `TC-AUTH-001` for Auth
- Derive the module code from the workflow's primary module in product_context.md

Each test case must have:
| TC-ID | Workflow | Scenario | Preconditions | Steps | Expected Result | Priority | Risk | BR Reference |

**Coverage target per workflow:**
- At minimum: 1 Functional test per workflow step + 1 Negative test per validation rule + 1 Boundary test per numeric/size constraint + 1 BR test per applicable business rule
- Do not skip workflows or rules — generate tests for everything found in the knowledge files

Write to `outputs/product_scan/product_test_suite.md` using this structure:

```markdown
# Full Product Test Suite
**Generated:** [date] | **Source:** product_context.md + business_rules.md

## Coverage Summary
| Workflow | Functional | Negative | Boundary | BR Tests | Total |
|----------|------------|----------|----------|----------|-------|
| Buy Policy | N | N | N | N | N |
| Submit Claim | N | N | N | N | N |
| Policy Renewal | N | N | N | N | N |
| Agent Portal | N | N | N | N | N |
| Admin | N | N | N | N | N |
| **Total** | | | | | **N** |

## Workflow: Buy Policy
[Test cases table]

## Workflow: Submit Claim
[Test cases table]

## Workflow: Policy Renewal
[Test cases table]

## Workflow: Agent Portal
[Test cases table]

## Workflow: Admin
[Test cases table]
```

---

### Step 4: Generate — Missing Integration Report

Review all cross-module data flows defined in product_context.md and business_rules.md.

For each integration point, determine:
- **Source Module** → **Target Module**
- **Data / Event flowing** (e.g., "claim approval triggers payment")
- **Has test coverage?** — YES if a test case in Step 3 covers this flow, NO otherwise
- **Risk Level** — based on financial impact, data mutation, user impact

Flag all integration points with no test coverage as **gaps**.

Write to `outputs/product_scan/missing_integrations.md` using this structure:

```markdown
# Missing Integration Report
**Generated:** [date] | **Source:** knowledge base only

## Integration Coverage Summary
- Total integration points identified: N
- Covered by test suite: N
- **Gaps found: N** ← this is the key number

## All Integration Points

| # | Source Module | Target Module | Data / Event | Test Coverage | Risk |
|---|--------------|---------------|--------------|---------------|------|
| 1 | Claims | Payment Gateway | Claim payout trigger | ❌ Not covered | 🔴 High |
| 2 | Auth | MFA Service | OTP on login | ✅ Covered (TC-AUTH-006) | 🔴 High |
| ... | | | | | |

## Gaps — Integration Points With No Test Coverage
[List each gap with recommended test case to add]

## Recommendations
[Top 5 integration areas to prioritize in the next sprint]
```

---

### Step 5: Save Output Files
Ensure these three files exist with the generated content:
- `outputs/product_scan/module_risk_map.md`
- `outputs/product_scan/product_test_suite.md`
- `outputs/product_scan/missing_integrations.md`

---

### Step 6: Generate HTML Report
Create a self-contained HTML report at `outputs/product_scan/product_qa_report.html`.

The HTML file must:
- Be fully self-contained (all CSS inline, no external dependencies)
- Use a dark professional theme (`#0d1117` background, `#161b22` cards)
- Open directly in any browser without a server

**Structure of the HTML report:**

```
<header>
  QA Brain branding
  Title: "Product QA Scan — [Product Name]"
  Subtitle: "Generated from knowledge base · No story required"
  Date + ✅ Scan Complete badge

<summary cards row> — 4 cards:
  1. Total Test Cases (count across all workflows) — blue
  2. High Risk Modules (count of modules ≥ 7.0) — red
  3. Integration Gaps (count of uncovered integration points) — orange
  4. Business Rules Covered (count of business rules referenced in tests) — green

<Module Risk Map section — full width>
  - Horizontal risk bars for each module, sorted highest to lowest
  - Color: ≥7 = red, 5-6.9 = yellow/orange, <5 = green
  - Each bar shows module name + 4 dimension scores + overall

<Test Coverage section — full width>
  - Summary table: workflow × category (Functional/Negative/Boundary/BR Tests)
  - Tabbed view: one tab per workflow, showing test cases table
  - TC-ID | Scenario | Priority | Risk | BR Reference columns

<Integration Gaps section — full width>
  - Two sub-sections: Covered ✅ and Not Covered ❌
  - Table: Source → Target | Data/Event | Risk Level
  - Highlighted gap count badge at top

<footer>
  "Generated by QA Brain · github.com/bavithiranhardy14/qa-brain"
```

---

## Final Summary to User
After completing all steps, present:
- **Product scanned:** [product name from product_context.md]
- **Modules analysed:** count
- **Highest risk module:** name + score
- **Total test cases generated:** count
- **Integration gaps found:** count
- **Business rules referenced:** count
- **Files created:**
  - `outputs/product_scan/module_risk_map.md`
  - `outputs/product_scan/product_test_suite.md`
  - `outputs/product_scan/missing_integrations.md`
  - `outputs/product_scan/product_qa_report.html`
