---
name: review-story
description: 'Review a user story for ambiguity, missing acceptance criteria, and testability issues. Use when: a story is about to enter testing, you need to check story quality, identifying gaps before test case generation, producing an execution summary.'
---

# Review Story

## When to Use
- A story has been written and needs QA review before testing begins
- You want to catch ambiguities or missing acceptance criteria early
- You need to produce an execution summary after test generation

## Procedure

### Step 0: Check the Story Index
Read [story_index.md](../../stories/story_index.md) to confirm which story is 🔄 In Progress.
The active story must be loaded in `stories/current_story.md` before proceeding.

### Step 1: Read the Story
Read the current story:
- [Current Story](../../stories/current_story.md)

### Step 2: Check Story Quality
Evaluate against these criteria:

**Ambiguity Check**
- Are any acceptance criteria vague or open to multiple interpretations?
- Do any steps lack a defined expected outcome?
- Are roles and permissions clearly specified?

**Completeness Check**
- Is there at least one AC per stated user goal?
- Are error/failure scenarios addressed in the ACs?
- Are edge cases and boundary conditions mentioned?

**Testability Check**
- Can each AC be verified with a specific, repeatable test?
- Are expected results measurable and observable?
- Are dependencies and preconditions documented?

### Step 3: Count Test Coverage
If `outputs/generated_test_cases.md` has been populated, count the tests by category:
- Functional Tests
- Negative Tests
- Boundary Tests
- Integration Tests

### Step 4: Identify Missing Areas
List any coverage gaps — ACs with no corresponding test, risky flows not tested, or assumptions not validated.

### Step 5: Write Output
Overwrite [outputs/execution_summary.md](../../outputs/execution_summary.md) with:
- Story name and ID
- Test count by category

The execution_summary must include:
- Story name and ID
- Coverage summary
- Missing areas
- Story quality notes (ambiguities, missing ACs, testability issues)
- Recommendations
- Generation timestamp

### Step 6: Save to Story Outputs
Read [story_index.md](../../stories/story_index.md) to get the current Story ID that is 🔄 In Progress.
Create `stories/outputs/US-NNN_execution_summary.md` (replacing NNN with the actual ID) with the same content just written to `outputs/execution_summary.md`.

## Notes
- This skill is the first step in the MVP workflow — run it before `analyze-risk` and `generate-test-cases`
- It can also be run last to produce the final execution summary after test generation
- Flag ambiguous ACs as blocking if they would lead to incorrect test cases being written
