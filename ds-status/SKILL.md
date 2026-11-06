---
name: ds-status
description: Reads STATUS.md from an active design system project, shows the current progress with a percentage, and lets you mark items done through a conversation. Keeps progress legible across sessions without opening files manually.
---

# Design System Progress Tracker

A bookkeeping skill. It does ONE thing: read the `STATUS.md` of a design system project, show where things stand, and let you update it without opening the file.

The value is cross-session continuity. Design systems are built across many sessions. Without a persistent tracker, every session starts with "where were we?" — the STATUS.md is the answer.

## Hard Rules

1. **Read before displaying.** Always read the actual `STATUS.md` before showing progress. Never assume what is done.
2. **Targeted edits only.** When updating, change only the lines the user confirmed. Do not rewrite the file, re-sort sections, or change formatting beyond the checkbox.
3. **Fuzzy matching on names.** Users will say "done the buttons" not "Buttons (primary, secondary, ghost, destructive, icon button)". Match on the most significant word(s) and confirm the match before marking done.
4. **Recompute percentage from actual checkboxes.** Count `[x]` vs `[ ]` lines. Never estimate or approximate.

## Workflow

### Phase 1 — Locate the project

Look for `STATUS.md` in this order:
1. Current working directory
2. Parent directory (if the cwd is a subfolder of the project)

If not found, ask: **"Which design system project do you want to check? I'll look for its STATUS.md."**

### Phase 2 — Display current progress

Read `STATUS.md`, then present:

```
{project-name} — Design System
{date from file or today}

VISUAL FOUNDATIONS         {n}/{total} done
  ✅ Color palette
  ✅ Typography
  ⬜ Spacing scale
  ⬜ Border radii
  ...

CORE COMPONENTS            {n}/{total} done
  ✅ Buttons
  ⬜ Inputs
  ...

PLATFORM COMPONENTS        {n}/{total} done
  ⬜ {component}
  ...

──────────────────────────────
Overall: {X} of {total} items — {percentage}%
```

Compute `{percentage}` from actual checkbox counts. Round to the nearest integer.

### Phase 3 — Ask what changed

After showing the status:

> What got done? List component names or say "finished [section name]" to mark a whole section.

Wait for the reply.

### Phase 4 — Update STATUS.md

For each item the user mentions:

1. Find the matching line using fuzzy matching on key words (case-insensitive).
2. If a single match: mark it done immediately.
3. If multiple possible matches: list them and ask which. Do not guess.
4. If the item does not exist in the file: add it under the most logical section, marked done.

Apply edits as targeted replacements — one per changed line. Never rewrite the whole file.

After all edits:
- Recompute overall percentage
- If all items in a section are done, add `✓` after the section heading
- If overall percentage reaches 100%, update the status header to `🟢 Complete`

### Phase 5 — Confirm and suggest next step

Reply based on the new state:

```
STATUS.md updated. {project-name} is now {X}% done ({n} of {total} items).
```

Then one of:

- If items remain: `{n} items left: {list the remaining ones}. Work on the next with /ds-component.`
- If foundations and core components are done but platform components remain: `Foundations and core components are complete — good time to start building screens with /ds-screen.`
- If 100% complete: `The {project-name} design system is complete.`
