---
name: reset-product
description: 'Clear all product-specific outputs and stories to prepare the workspace for a new product. Resets story index, clears all outputs, and preserves skills and knowledge file structure. Use when: switching to a new product, onboarding a new client project, starting fresh with new knowledge files.'
argument-hint: 'Optionally provide the new product name to pre-label the reset, or leave blank'
---

# Reset Product — Clear Workspace for a New Product

## When to Use
- You are switching qa-brain to a different product
- A new client's `product_context.md` and `business_rules.md` are ready to load
- You want to wipe all previous outputs and stories without touching skills

## What This Skill Does
1. Deletes all files inside `outputs/` (keeps the folder)
2. Deletes all files inside `outputs/product_scan/` (keeps the folder)
3. Deletes all story files (`stories/US-NNN_*.md`)
4. Clears `stories/current_story.md` content
5. Deletes all files inside `stories/outputs/` (keeps the folder)
6. Resets `stories/story_index.md` — Next Story ID back to US-001, clears story rows
7. Confirms what was cleared and what to do next

## What This Skill Does NOT Touch
- `.github/skills/` — all skills are product-agnostic, kept as-is
- `knowledge/product_context.md` — you replace this manually with the new product's file
- `knowledge/business_rules.md` — you replace this manually with the new product's file
- `README.md`, `.gitignore`

---

## Procedure

### Step 1: Confirm Before Clearing
Before doing anything, show the user this confirmation message:

```
⚠️ Reset Product — This will permanently delete:

  - All files in outputs/
  - All files in outputs/product_scan/
  - All US-NNN story files in stories/
  - All files in stories/outputs/
  - Contents of stories/current_story.md
  - All story rows in stories/story_index.md

Skills and knowledge files will NOT be touched.

Type CONFIRM to proceed, or anything else to cancel.
```

Wait for the user's response. Only proceed if they type `CONFIRM` (case-insensitive). If anything else, stop and do nothing.

### Step 2: Clear outputs/
Delete all files inside `outputs/` folder.
If `outputs/product_scan/` exists, delete all files inside it too.
Keep both folders — do not delete the folders themselves.

### Step 3: Clear stories/outputs/
Delete all files inside `stories/outputs/`.
Keep the folder.

### Step 4: Delete Story Files
Delete all files matching `stories/US-*.md`.

### Step 5: Clear current_story.md
Overwrite `stories/current_story.md` with:

```markdown
# Current Story

_No active story. Run `/new-story` to begin._
```

### Step 6: Reset story_index.md
Overwrite `stories/story_index.md` with:

```markdown
# Story Index

> **Next Story ID: US-001**

## Stories

| Story ID | Title | Sprint | Status | Story File | Outputs |
|----------|-------|--------|--------|------------|---------|

_No stories yet. Run `/new-story` to process your first story._
```

### Step 7: Confirm Completion
Show the user:

```
✅ Workspace reset complete.

Cleared:
  - outputs/ — all files deleted
  - outputs/product_scan/ — all files deleted
  - stories/US-*.md — all story files deleted
  - stories/outputs/ — all files deleted
  - stories/current_story.md — reset to empty
  - stories/story_index.md — reset to US-001

Next steps:
  1. Replace knowledge/product_context.md with your new product's context
  2. Replace knowledge/business_rules.md with your new product's business rules
  3. Run /scan-product to generate the full product QA scan
  4. Or run /new-story to start processing individual stories
```
