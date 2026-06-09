---
name: extract-knowledge
description: 'Scan the current codebase and generate product_context.md and business_rules.md knowledge files. Use when: onboarding a new project into qa-brain, a developer shares their codebase, you need to auto-generate knowledge files from source code instead of writing them manually.'
argument-hint: 'Optionally specify subfolder to scan (e.g. api/ or frontend/) or leave blank to scan full project'
---

# Extract Knowledge — Generate Knowledge Files from Codebase

## When to Use
- A developer has shared their codebase and you need to populate the knowledge files
- You are onboarding a new product into qa-brain
- `knowledge/product_context.md` or `knowledge/business_rules.md` are empty or missing

## What This Skill Does
1. Scans all source files in the workspace
2. Extracts modules, workflows, roles, and tech stack → writes `knowledge/product_context.md`
3. Extracts business rules, validations, constraints, and edge cases → writes `knowledge/business_rules.md`

## What It Does NOT Do
- Does not invent rules — only extracts what exists in the code
- Does not overwrite skills or outputs

---

## Procedure

### Step 1: Locate the Source Code
Check the workspace structure:
- If a subfolder argument was provided (e.g. `api/`, `src/`), scope the scan to that folder
- If no argument, scan the entire workspace
- Identify the project type: look for `package.json`, `requirements.txt`, `pom.xml`, `build.gradle`, `*.csproj`, `Gemfile`, `go.mod`

### Step 2: Extract for product_context.md

Scan all source files and extract:

**Application name and type:**
- Look in: `package.json` (name field), `README.md`, `app.config.*`, `manifest.*`, `settings.py`, `application.properties`

**User roles:**
- Look in: auth middleware, role enums, permission files, database seeders, constants files, decorator names (`@role`, `@permission`, `hasRole(...)`)

**Modules / feature areas:**
- Look in: routes files, controllers folder, page components, navigation config, sidebar config, API endpoint prefixes (`/api/claims`, `/api/policies`)
- Each distinct route group or feature folder = one module

**User workflows:**
- For each module found, trace the full end-to-end flow:
  - Entry point: route or page component
  - Through: controller/service/handler
  - To: final outcome (DB write, API response, redirect, notification)
- Write each flow as numbered steps in plain English

**Tech stack:**
- Frontend: look in `package.json` dependencies (React, Angular, Vue, Next.js, etc.)
- Backend: look in `package.json`, `requirements.txt`, `pom.xml` (Express, Django, Spring, FastAPI, etc.)
- Database: look in ORM config, connection strings, `.env.example` (PostgreSQL, MySQL, MongoDB, etc.)
- Auth: look for JWT, OAuth2, Passport, Spring Security, Devise, etc.
- Integrations: look for third-party SDK imports, `.env` keys (Stripe, Twilio, SendGrid, S3, etc.)

Write the output to `knowledge/product_context.md`:

```markdown
# Product Context

## Application Overview
**Application Name:** [extracted]
**Type:** [Web App / REST API / Mobile / etc.]
**Primary Users:** [extracted roles]

[2-3 line description of what the app does based on routes and models found]

## Modules
| Module | Description | Primary Roles |
|--------|-------------|---------------|
[one row per module/feature area found]

## User Workflows
### Workflow 1: [Name]
1. [step]
2. [step]
[one workflow section per major feature]

## Tech Stack
- Frontend: [extracted or "Not found in codebase"]
- Backend: [extracted]
- Database: [extracted]
- Auth: [extracted]
- Integrations: [extracted or "Not found in codebase"]
```

---

### Step 3: Extract for business_rules.md

Scan all source files and extract:

**Business rules:**
- Look in: service layer, controllers, validators, middleware, `if/else` blocks that enforce limits or eligibility
- Examples: age checks, balance checks, approval flows, status transitions, financial calculations
- Write each as plain English: "A user must be X to do Y"

**Validation rules:**
- Look in: form validators, request/DTO validators, schema definitions (Joi, Zod, Yup, Pydantic, Bean Validation), model constraints
- Capture: field name + condition + exact error message if present

**System constraints:**
- Look in: constants files, config files, `.env.example`, environment variables
- Examples: max file size, token expiry, rate limits, max items per request, timeout values

**Edge cases:**
- Look in: `try/catch` blocks handling specific business conditions, `TODO`/`FIXME` comments, special-case `if` blocks, transaction rollback logic, duplicate detection logic

Write the output to `knowledge/business_rules.md`:

```markdown
# Business Rules

## Business Rules
| ID | Rule | Module | Priority |
|----|------|--------|----------|
| BR-01 | [plain English rule extracted from code] | [module] | High/Medium/Low |
[number sequentially — include all rules found]

## Validation Rules
| Field | Rule | Error Message |
|-------|------|---------------|
[one row per validation — use "Not specified in code" if no error message found]

## System Constraints
- [constraint extracted from code]
[one bullet per constraint found]

## Edge Cases
- [edge case extracted from code]
[one bullet per edge case found]
```

---

### Step 4: Save Both Files
Write the completed content to:
- `knowledge/product_context.md`
- `knowledge/business_rules.md`

---

### Step 5: Confirm Completion
Show the user:

```
✅ Knowledge extraction complete.

Files written:
  - knowledge/product_context.md
  - knowledge/business_rules.md

Summary:
  - Modules found: N
  - Workflows documented: N
  - Business rules extracted: N
  - Validation rules extracted: N
  - System constraints found: N

Next steps:
  - Review both files and add anything the scanner may have missed
  - Run /scan-product to generate the full QA scan
  - Or run /new-story to start processing individual stories
```

## Extraction Rules
- Do NOT invent rules — only extract what actually exists in the code
- If a rule is implied by logic (`if (age < 18) reject`) write it as plain English
- If a section has nothing, write `Not found in codebase` — do not skip the section
- Scan all folders including test files — tests often reveal business rules not visible in source
- If a rule appears in multiple places, write it once
