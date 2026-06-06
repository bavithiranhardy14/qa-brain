---
name: analyze-risk
description: 'Analyze risk for a user story and identify testing priorities. Use when: assessing story risk before testing, identifying high-risk areas, determining where to focus test effort, generating a risk analysis report.'
---

# Analyze Risk

## When to Use
- Before or after generating test cases for a story
- You need to prioritize test effort based on business impact
- The story touches sensitive modules (auth, payments, data, integrations)

## Procedure

### Step 0: Check the Story Index
Read [story_index.md](../../stories/story_index.md) to confirm which story is 🔄 In Progress.
The active story must be loaded in `stories/current_story.md` before proceeding.

### Step 1: Read Input Files
Read all three files before scoring:
- [Product Context](../../knowledge/product_context.md)
- [Business Rules](../../knowledge/business_rules.md)
- [Current Story](../../stories/current_story.md)

### Step 2: Assess Risk Factors
Score each dimension from 1 (low) to 10 (high):

| Dimension | Consider |
|-----------|---------|
| **Data Integrity** | Does this story write, update, or delete data? |
| **Security** | Does this story touch auth, permissions, or PII? |
| **Performance** | Does this story involve bulk operations or high-traffic flows? |
| **Integration** | Does this story depend on or affect other modules or systems? |
| **Business Logic** | How many business rules apply? How complex are they? |
| **User Experience** | Can a bad outcome directly impact users or cause data loss? |

### Step 3: Calculate Overall Risk
- **Overall Risk Score** = weighted average of dimension scores
- **Risk Level**:
  - 1–3: Low
  - 4–6: Medium
  - 7–8: High
  - 9–10: Critical

### Step 4: Identify High Risk Areas
For each area scoring 7+, document:
- The specific area
- Why it is high risk
- Recommended test focus

### Step 5: Write Output
Overwrite [outputs/risk_analysis.md](../../outputs/risk_analysis.md) with the completed risk analysis, including score, breakdown, high risk areas, and mitigation suggestions.

### Step 6: Save to Story Outputs
Read [story_index.md](../../stories/story_index.md) to get the current Story ID that is 🔄 In Progress.
Create `stories/outputs/US-NNN_risk_analysis.md` (replacing NNN with the actual ID) with the same content just written to `outputs/risk_analysis.md`.

## Notes
- Always cross-reference business rules when scoring Business Logic dimension
- Regulatory requirements automatically raise the Security score
- A single Critical area elevates the overall Risk Level to at least High
