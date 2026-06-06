---
name: new-story
description: 'Run the full QA Copilot pipeline for a new user story. Use when: you have a new user story to process, you want to go from story input to complete test cases in one step, running the full review → risk → generate workflow automatically.'
argument-hint: 'Paste or describe the user story to process'
---

# New Story — Full QA Pipeline

## When to Use
- You have a new user story (pasted in chat, or described)
- You want the full QA workflow to run automatically end-to-end
- No manual file copying, no manual index updates

## What This Skill Does Automatically
1. Assigns the next Story ID from `story_index.md`
2. Creates `stories/US-NNN_title.md` with the story content
3. Registers the story in `story_index.md` as 🔄 In Progress
4. Writes the story into `stories/current_story.md`
5. Runs **Review Story** → flags ambiguities and missing ACs
6. Runs **Analyze Risk** → scores risk and identifies high-risk areas
7. Runs **Generate Test Cases** → produces functional, negative, boundary, integration tests
8. Saves all three outputs to `stories/outputs/US-NNN_*.md`
9. Appends **QA Pipeline Results** section into the story file
10. Updates `story_index.md` to ✅ Complete with output links
11. Generates a **self-contained HTML report** at `stories/outputs/US-NNN_qa_report.html`

---

## Procedure

### Step 1: Read the Story Index
Read [story_index.md](../../stories/story_index.md).
- Find the **Next Story ID** at the top of the file.
- Use that as the Story ID for this story (e.g., `US-003`).

### Step 2: Derive the Story File Name
- Take the story title, lowercase it, replace spaces with underscores, remove special characters.
- File name pattern: `US-NNN_title_words.md`
- Example: "Policy Renewal Reminder" → `US-003_policy_renewal_reminder.md`

### Step 3: Create the Story File
Create `stories/US-NNN_title.md` with the full story content provided, using this structure:

```markdown
# US-NNN — [Story Title]

## Story Details
**Story ID:** US-NNN
**Title:** [Title]
**Epic:** [Epic if known, else TBD]
**Sprint:** [Sprint if known, else TBD]
**Status:** 🔄 In Progress

## User Story
> As a [role], I want to [action], So that [benefit].

## Acceptance Criteria
- [ ] AC-1: ...

## Assumptions
- ...

## Out of Scope
- ...

## Dependencies
- ...

## Notes
- ...
```

### Step 4: Update story_index.md
- Increment **Next Story ID** by 1.
- Add a new row to the Stories table:
  `| US-NNN | [Title] | [Sprint] | 🔄 In Progress | [link to story file] | — |`

### Step 5: Write current_story.md
Overwrite [current_story.md](../../stories/current_story.md) with the full content of the new story file.

### Step 6: Run — Review Story
Read:
- [current_story.md](../../stories/current_story.md)

Check for:
- Ambiguity in acceptance criteria
- Missing AC for stated goals
- Testability issues (vague expected results, missing preconditions)
- Missing error/edge cases

Write findings to [outputs/execution_summary.md](../../outputs/execution_summary.md).

### Step 7: Run — Analyze Risk
Read:
- [knowledge/product_context.md](../../knowledge/product_context.md)
- [knowledge/business_rules.md](../../knowledge/business_rules.md)
- [current_story.md](../../stories/current_story.md)

Score risk dimensions (1–10): Data Integrity, Security, Performance, Integration, Business Logic, User Experience.
Calculate overall score and risk level (Low / Medium / High / Critical).
Identify areas scoring 7+ as high risk with recommended test focus.

Write findings to [outputs/risk_analysis.md](../../outputs/risk_analysis.md).

### Step 8: Run — Generate Test Cases
Read:
- [knowledge/product_context.md](../../knowledge/product_context.md)
- [knowledge/business_rules.md](../../knowledge/business_rules.md)
- [current_story.md](../../stories/current_story.md)

Generate test cases in four categories:
1. **Functional Tests** — one per acceptance criterion (happy path)
2. **Negative Tests** — invalid inputs, unauthorized actions, failure paths
3. **Boundary Tests** — min/max values, empty inputs, limit conditions
4. **Integration Tests** — cross-module data flows and dependencies

Each test case must have: TC-ID, Scenario, Preconditions, Steps, Expected Result, Priority, Risk.

Write to [outputs/generated_test_cases.md](../../outputs/generated_test_cases.md).

### Step 9: Save Outputs to Story Outputs Folder
Create the following files with the content just generated:
- `stories/outputs/US-NNN_execution_summary.md` ← copy of outputs/execution_summary.md
- `stories/outputs/US-NNN_risk_analysis.md` ← copy of outputs/risk_analysis.md
- `stories/outputs/US-NNN_test_cases.md` ← copy of outputs/generated_test_cases.md

### Step 10: Write QA Pipeline Results into the Story File
Append a `## QA Pipeline Results` section to `stories/US-NNN_title.md` with the following structure:

```markdown
---

## QA Pipeline Results

**Pipeline run:** [today's date] | **Status:** ✅ Complete

### 🔍 Review Story

| Check | Result |
|-------|--------|
| ACs clear? | [result] |
| Ambiguities | [count + summary] |
| Missing ACs | [count + list] |
| Testability | [result] |

📄 [View Execution Summary](outputs/US-NNN_execution_summary.md)

---

### ⚠️ Analyze Risk

| Dimension | Score |
|-----------|-------|
| Security | X / 10 |
| Business Logic | X / 10 |
| Integration | X / 10 |
| Data Integrity | X / 10 |
| User Experience | X / 10 |
| Performance | X / 10 |
| **Overall** | **X / 10 — [🔴 High / 🟡 Medium / 🟢 Low]** |

**Top risks:** [brief list]

📄 [View Risk Analysis](outputs/US-NNN_risk_analysis.md)

---

### ✅ Generate Test Cases

| Category | Count |
|----------|-------|
| Functional | N |
| Negative | N |
| Boundary | N |
| Integration | N |
| **Total** | **N** |

📄 [View Test Cases](outputs/US-NNN_test_cases.md)
```

### Step 11: Update story_index.md to Complete
Update the row added in Step 4:
- Change status from 🔄 In Progress → ✅ Complete
- Add links to the three output files in the Outputs column:
  `[Tests](outputs/US-NNN_test_cases.md) · [Risk](outputs/US-NNN_risk_analysis.md) · [Summary](outputs/US-NNN_execution_summary.md)`

### Step 12: Generate HTML QA Report
Create a self-contained HTML report at `stories/outputs/US-NNN_qa_report.html` that consolidates all three outputs into one shareable page.

The HTML file must:
- Be fully self-contained (all CSS inline — no external dependencies)
- Use a dark professional theme (`#0d1117` background, `#161b22` cards)
- Open directly in any browser without a server

**Structure of the HTML report:**

```
<header>
  QA Brain branding + Story ID + Title + Epic + Sprint + Date + ✅ Pipeline Complete badge
</header>

<summary cards row> — 4 cards:
  1. Total Test Cases (count) — blue
  2. Overall Risk (X/10 + High/Med/Low label) — red/yellow/green
  3. Story Quality Gaps (missing AC count) — yellow
  4. Ambiguities (count) — orange

<two-column grid>
  LEFT — Risk Analysis panel:
    - Overall risk score badge (🔴/🟡/🟢)
    - Horizontal score bars for each of the 6 dimensions, color-coded:
        9-10 = red, 7-8 = orange, 5-6 = blue, 1-4 = green
    - Each bar shows label + filled bar + numeric score

  RIGHT — Story Review panel:
    - Quality checklist (ACs clear, ambiguities, missing ACs, testability)
    - Each item with ✅ or ⚠️ icon + short note

  LEFT — Missing Acceptance Criteria panel:
    - Bulleted list with 🔴 Security or 🟡 Warning tag + description

  RIGHT — Recommended Test Focus panel:
    - Numbered list of top focus areas with brief explanation

<full-width> — Test Cases panel:
  - 4 tabs: Functional | Negative | Boundary | Integration (with counts)
  - Each tab shows a table: TC-ID | Scenario | Preconditions | Steps | Expected Result | Priority | Risk
  - Priority badges: High=red, Medium=yellow, Low=green
  - Risk badges: High=red pill, Medium=yellow pill, Low=green pill
  - Tab switching via inline JavaScript (no frameworks)

<full-width> — Recommendations panel:
  - Numbered list with blue circle counters

<footer>
  "Generated by QA Brain · github.com/bavithiranhardy14/qa-brain"
```

Populate all values from the data generated in Steps 6, 7, and 8.
Use the actual story title, risk scores, test case count, AC gaps, and all 24 (or N) test case rows.

---

## Final Summary to User
After completing all steps, present a brief summary:
- **Story ID & Title**
- **Risk Level** (overall score)
- **Test Cases Generated** (count by category)
- **Story Quality Issues Found** (count of ambiguities / missing ACs)
- **Files Created** (list all 5 files: story file + 3 outputs + HTML report)
- **HTML Report:** `stories/outputs/US-NNN_qa_report.html` — open in browser for a shareable one-page view
