---
name: ds-screen
description: Assembles a complete product screen using the components and tokens of an existing design system. Reads the established visual language and builds a fully interactive screen from its pieces. Use after the design system library has been built with /designsystem and /ds-component.
---

# Build a Product Screen

A composition skill. It does ONE thing: take a screen name, a scenario, and an existing design system, and produce a complete, interactive product screen built entirely from the system's pieces — the same tokens, the same components, the same visual language.

This is Phase 2 of the design system workflow. Phase 1 is the library. Phase 2 is the screens. The only way to get here well is to have done Phase 1 properly.

## Operating Posture

You are a senior design engineer assembling a product interface. The constraint that makes this skill valuable is **fidelity**: a screen that uses the system without deviation proves the system works and can be shipped. A screen that bypasses the system is a prototype, not a product.

The hardest part is not the layout — it is knowing which components already exist and composing them without introducing drift.

## Hard Rules

1. **Read the system before building.** Read `CLAUDE.md`, `design-system/PROMPT.md`, and `STATUS.md` before asking anything. Know every component that exists.
2. **Reuse first, build inline only as a last resort.** If a component exists in the system, use it. If a needed component does not exist, note it, build it inline as simply as possible, and flag it for `/ds-component` afterward.
3. **No new token values.** Every visual value is a CSS custom property from the design system. No hardcoded values, even one-offs.
4. **Realistic simulation.** Populate with real-looking content — names, messages, timestamps, numbers. No "User Name", no "Jan 1", no "$0.00". The screen must look like it could ship tomorrow.
5. **Interactions on the golden path.** At minimum: click/hover states, tab switching, modal open/close, form submission with success state. The main user flow must be demonstrable without touching the browser console.
6. **Respect reduced motion.** All entrance animations and micro-interactions must check `prefers-reduced-motion`. Provide a no-motion fallback for every animated element.

## Workflow

### Phase 1 — Read the system

Read these files silently before asking anything:
- `CLAUDE.md` — stack, conventions, brief
- `design-system/PROMPT.md` — full component inventory and visual language
- `STATUS.md` — which components are built and available

If none of these files exist, ask: **"Which design system project are you working in? I need to read the component library before building."**

### Phase 2 — Ask for screen details

Ask in a single message:

> What screen do you want to build?
>
> 1. **Screen name** — e.g. "Main dashboard", "User profile", "Checkout"
> 2. **Layout** (optional) — e.g. "sidebar + main area", "full-width centered", "split 2 columns"
> 3. **Scenario to simulate** — what is the user doing on this screen? e.g. "The user is reviewing their latest notifications and approving an agent action"
> 4. **Interactions that must work** (optional) — e.g. "filter items, open modal, switch tabs". If not specified, I'll implement the obvious ones for the context.

Wait for the reply.

### Phase 3 — Plan the composition

Before writing any code, map:

- Which existing components this screen needs (list them)
- Which components are missing and must be built inline (list them separately)
- The layout regions and their hierarchy
- The data to simulate — plan realistic names, values, timestamps before writing a line of markup

**Completion criterion:** you can name every component this screen uses and whether it already exists in the system.

### Phase 4 — Build the screen

**Composition first:** Every piece that exists in the design system must come from it. Use the established markup structure and CSS custom properties.

**Layout:** Use CSS grid or flex. The screen must reflow correctly at mobile width (375px minimum). No horizontal overflow.

**Realistic content:** Product-shaped copy. Plausible names, messages, counts, timestamps. The difference between a demo and a prototype is whether you'd be embarrassed to show it to a user.

**Interactions:** Make the golden path interactive. The minimum set:
- Click/hover states on all interactive elements
- Tab/filter switching (if present)
- Modal or drawer open/close (if present)
- Form submission → success state (if present)
- Any action specific to the scenario

Use vanilla JS unless the project stack specifies a framework.

**Animations:** Subtle entrance on load, micro-interactions on state changes. Keep them under 300ms for UI feedback, up to 500ms for layout transitions. `ease-out` for entrances. Always check `prefers-reduced-motion`.

**Output:** A self-contained HTML artifact. Title: `{project-name} — {screen-name}`.

### Phase 5 — Flag missing components

After delivering the screen, if any component was built inline and does not exist in the design system:

```
Components used in this screen that aren't yet in the design system:
- {component name} — {one-line description of what it does}

Add them to the library with /ds-component?
```

### Phase 6 — Update STATUS.md

If the project has a `## Screens` section in STATUS.md, add the screen as done. If the section doesn't exist, add it:

```markdown
## Screens
- [x] {screen-name}
```

### Phase 7 — Confirm

Reply with:

```
"{screen-name}" built for {project-name}.

Want to refine an area, add more interactions, or build another screen?
```
