---
name: generate-test-cases
description: 'Generate project-aware test cases for a user story. Use when: a story is ready for QA, generating functional/negative/boundary/integration test cases, writing test cases from acceptance criteria.'
---

# Generate Test Cases

## When to Use
- A user story has been added to `stories/current_story.md`
- You need to generate structured test cases using project knowledge
- You want thorough coverage beyond generic test cases

## Procedure

### Step 0: Check the Story Index
Read [story_index.md](../../stories/story_index.md) to confirm which story is 🔄 In Progress.
The active story must be loaded in `stories/current_story.md` before proceeding.

### Step 1: Read Input Files
Read all three files before generating anything:
- [Product Context](../../knowledge/product_context.md)
- [Business Rules](../../knowledge/business_rules.md)
- [Current Story](../../stories/current_story.md)

### Step 2: Analyse
- Map each acceptance criterion to test scenarios
- Identify which business rules and validation rules apply
- Spot integration touchpoints with other modules
- Find boundary conditions from validation rules and constraints

### Step 3: Generate Test Cases
Produce test cases in four categories:
1. **Functional Tests** — Happy path for each acceptance criterion
2. **Negative Tests** — Invalid inputs, unauthorized actions, failure paths
3. **Boundary Tests** — Min/max values, empty inputs, limit conditions
4. **Integration Tests** — Data flow between modules, external dependencies

Each test case must include:
- **TC-ID**: `TC-[Module]-[NNN]`
- **Scenario**: One clear line describing what is tested
- **Preconditions**: What must be true before execution
- **Steps**: Numbered, specific steps
- **Expected Result**: Verifiable, specific outcome
- **Priority**: High / Medium / Low
- **Risk**: High / Medium / Low

### Step 4: Write Output
Overwrite [outputs/generated_test_cases.md](../../outputs/generated_test_cases.md) with the generated test cases, preserving the four-category table structure.

### Step 5: Save to Story Outputs
Read [story_index.md](../../stories/story_index.md) to get the current Story ID that is 🔄 In Progress.
Create `stories/outputs/US-NNN_test_cases.md` (replacing NNN with the actual ID) with the same content just written to `outputs/generated_test_cases.md`.

### Step 6: Update story_index.md
In [story_index.md](../../stories/story_index.md), update the row for this story:
- If all three outputs now exist (test_cases, risk_analysis, execution_summary), set status to ✅ Complete
- Add output links: `[Tests](outputs/US-NNN_test_cases.md) · [Risk](outputs/US-NNN_risk_analysis.md) · [Summary](outputs/US-NNN_execution_summary.md)`

## Quality Checklist
Before finishing, verify:
- Every AC has at least one functional test
- At least one negative test per user-facing input
- Business rules referenced in test steps where relevant
- No generic tests — all scenarios reflect this product's terminology and rules
