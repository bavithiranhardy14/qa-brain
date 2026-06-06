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
9. Updates `story_index.md` to ✅ Complete with output links

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

### Step 10: Update story_index.md to Complete
Update the row added in Step 4:
- Change status from 🔄 In Progress → ✅ Complete
- Add links to the three output files in the Outputs column:
  `[Tests](outputs/US-NNN_test_cases.md) · [Risk](outputs/US-NNN_risk_analysis.md) · [Summary](outputs/US-NNN_execution_summary.md)`

---

## Final Summary to User
After completing all steps, present a brief summary:
- **Story ID & Title**
- **Risk Level** (overall score)
- **Test Cases Generated** (count by category)
- **Story Quality Issues Found** (count of ambiguities / missing ACs)
- **Files Created** (list all 4 files: story file + 3 outputs)
