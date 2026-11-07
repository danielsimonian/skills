---
name: ds-component
description: Adds a new component to an existing design system. Reads the established tokens and visual language, then generates a component that belongs — same foundations, correct states, composable. Use after /designsystem has scaffolded the project.
---

# Add a Design System Component

A composition skill. It does ONE thing: take a component name and context, read the design system already established, and produce a component that genuinely belongs to it — using its tokens, its states, its visual language. Not a standalone widget dropped into a folder.

## Operating Posture

You are a senior design engineer extending an existing design system. The constraint that makes this skill valuable is **coherence**: a component that uses the system's tokens and patterns is worth ten components that don't. Before writing a single line, understand the foundation you are building on.

The hardest part is not the component — it is making it feel inevitable inside this specific system.

## Hard Rules

1. **Read the system before building.** Before asking anything, read `CLAUDE.md`, `design-system/PROMPT.md`, and `STATUS.md` silently. Know the tokens, the stack, the visual language.
2. **No new token values.** Use only CSS custom properties already defined. If you discover a gap (a missing semantic color, a missing radius), name it, add it to the token sheet, and use it. Do not hardcode.
3. **Compose, don't duplicate.** If the component contains a button, an avatar, a badge — use the ones already in the system. Never rebuild a piece that exists.
4. **States are not optional.** Every component ships with: `default`, `hover`, `focus-visible`, `disabled`. Add `loading` and `error` where the component can have those states. Add `selected`/`active` if the component has selection behavior.
5. **Update STATUS.md.** Mark the component done in the checklist. If it's a new component not in the original list, add it.

## Workflow

### Phase 1 — Read the system

Read these files silently before asking anything:
- `CLAUDE.md` — stack, conventions, brief
- `design-system/PROMPT.md` — component inventory and visual language
- `STATUS.md` — what already exists

If none of these files exist, ask: **"Which design system project are you working in? I'll read the existing foundations before building."**

### Phase 2 — Ask for component details

Ask in a single message:

> What component do you want to add?
>
> 1. **Name** — e.g. "Notification card", "Empty state", "Progress bar"
> 2. **Variants** (optional) — e.g. "info, success, error, warning"
> 3. **States** (optional) — if not specified, I'll include: default, hover, focus, disabled
> 4. **Usage context** (optional) — where does this appear in the interface? This helps calibrate size and behavior.

Wait for the reply.

### Phase 3 — Build the component

**Token discipline:** Every visual value comes from a CSS custom property. No hardcoded colors, sizes, or radii.

**Composition:** If the component contains any existing piece (button, avatar, badge, tooltip), reference it — do not rebuild it.

**States:** Ship `default`, `hover`, `focus-visible`, `disabled` as a minimum. Add `loading`, `error`, `selected` where applicable. Each state must be visually distinct and testable.

**Accessibility:** `role`, `aria-label`, `aria-disabled`, `tabindex` where appropriate. The component must be keyboard-operable.

**Responsive:** Works at 320px minimum width. No horizontal overflow. Use `flex-wrap` or `min-width: 0` to handle long content.

**Realistic content:** No lorem ipsum. No "Title goes here". Use plausible, product-shaped copy in all examples.

**Stack:** Next.js (App Router) + TypeScript strict + Tailwind CSS + shadcn/ui. If the component has a shadcn/ui primitive (Button, Dialog, Select, Tooltip…), add it with `npx shadcn@latest add {name}` and restyle it with the system's tokens — never rebuild it from scratch. Use Tailwind classes mapped to the CSS custom properties, `cn()` for conditional classes, and `cva` for variants. No hardcoded values, no `any`.

**Output format:**
- The component file: `components/ui/{name}.tsx` for core primitives, `components/platform/{name}.tsx` for product-specific components. Typed props, variants via `cva`, interactions functional.
- A catalog entry in `app/page.tsx` showing all variants and states side by side.

### Phase 4 — Update STATUS.md

After generating the component:

1. Find the matching item in `STATUS.md` and mark it done: `- [ ]` → `- [x]`.
2. If it is a new component not in the original checklist, add it under the appropriate section marked done.

Use a targeted edit — do not rewrite the entire file.

### Phase 5 — Confirm

Reply with:

```
"{component name}" added to the design system.
STATUS.md updated.

Want to add another component, or start building a screen with /ds-screen?
```
