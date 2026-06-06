# QA Copilot

> AI-powered QA assistant that turns user stories into structured test cases, risk analysis, and story quality reviews — using project-specific knowledge, not generic prompts.

---

## What It Does

Most AI tools generate generic test cases. QA Copilot is different.

It reads your **product context** and **business rules** first, then generates test cases that reflect your actual application — the right modules, the real validation rules, the actual user roles.

**One command. Three outputs. Zero copy-paste.**

```
Paste your story → /new-story
         ↓
  Review Story    → flags ambiguities and missing ACs
  Analyze Risk    → scores risk by dimension, identifies high-risk areas  
  Generate Tests  → functional, negative, boundary, and integration tests
         ↓
  All outputs saved. Story index updated. Done.
```

---

## Outputs Per Story

| Output | What It Contains |
|--------|----------------|
| **Test Cases** | Functional, Negative, Boundary, Integration tests with TC-ID, Steps, Expected Result, Priority, Risk |
| **Risk Analysis** | Risk score (1–10), breakdown by dimension, high-risk areas, mitigation suggestions |
| **Execution Summary** | Story quality notes, missing ACs, ambiguities, coverage gaps, recommendations |

---

## Project Structure

```
qa-copilot/
├── knowledge/
│   ├── product_context.md      # App overview, modules, workflows, terminology
│   └── business_rules.md       # Business rules, validation rules, constraints, compliance
│
├── stories/
│   ├── story_index.md          # Master tracker — all stories with status and output links
│   ├── current_story.md        # Active story (skills read from here)
│   ├── US-NNN_title.md         # Individual story files
│   └── outputs/
│       ├── US-NNN_test_cases.md
│       ├── US-NNN_risk_analysis.md
│       └── US-NNN_execution_summary.md
│
├── outputs/                    # Working output files (overwritten per story)
│   ├── generated_test_cases.md
│   ├── risk_analysis.md
│   └── execution_summary.md
│
├── prompts/
│   └── test_generation_prompt.md
│
└── .github/
    └── skills/
        ├── new-story/          # Full pipeline: review → risk → generate → save
        ├── review-story/       # Story quality review
        ├── analyze-risk/       # Risk scoring by dimension
        └── generate-test-cases/# Test case generation
```

---

## Skills (Copilot Commands)

| Skill | Trigger | What It Does |
|-------|---------|-------------|
| **new-story** | `/new-story` | Runs the full pipeline end-to-end for a new story |
| **review-story** | `/review-story` | Reviews story quality — ambiguity, missing ACs, testability |
| **analyze-risk** | `/analyze-risk` | Scores risk across 6 dimensions, identifies high-risk areas |
| **generate-test-cases** | `/generate-test-cases` | Generates 4 categories of project-aware test cases |

---

## How To Use

### 1. Set Up Your Knowledge Base

Fill in your project details:
- `knowledge/product_context.md` — your app, modules, workflows, terminology
- `knowledge/business_rules.md` — business rules, validation rules, constraints

> The richer these files are, the more accurate and project-specific every output will be.

### 2. Submit a Story

Paste your user story into GitHub Copilot Chat and run `/new-story`:

```
/new-story

As a customer, I want to reset my password so that I can regain access 
to my account if I forget it.

AC-1: Customer can request a reset using their registered email
AC-2: Reset link expires after 30 minutes
AC-3: Customer cannot reuse their last 3 passwords
AC-4: Account remains locked until reset is completed
```

### 3. Review the Outputs

Check `stories/story_index.md` for links to all generated outputs.

---

## Example Output — US-003 Password Reset

**Risk Level:** 🔴 High (7/10)

| Category | Tests Generated |
|----------|----------------|
| Functional | 6 |
| Negative | 7 |
| Boundary | 6 |
| Integration | 5 |
| **Total** | **24** |

**Issues flagged before testing started:**
- No AC for unregistered email behaviour → email enumeration risk
- AC-4 ambiguous → "reset completed" undefined
- No rate limiting AC
- No password complexity AC on new password

---

## Sprint 1 Scope

This is a deliberate MVP. It does exactly three things well:

- ✅ Review story quality
- ✅ Analyze risk
- ✅ Generate test cases

**Out of scope (by design):**
- Defect repository
- Historical defect analysis
- Vector database / RAG / embeddings
- Playwright or API automation generation
- Bug prediction

> Build it. Use it on real stories. Find out what's missing. Then add the next thing.

---

## Requirements

- [VS Code](https://code.visualstudio.com/)
- [GitHub Copilot](https://github.com/features/copilot) with Agent mode enabled
- No installs. No dependencies. No configuration beyond your knowledge files.

---

## Built With

- GitHub Copilot Agent Skills (`.github/skills/`)
- Markdown knowledge files as project context
- Zero code — pure prompt engineering and workflow design
