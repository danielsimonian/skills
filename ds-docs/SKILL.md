---
name: ds-docs
description: Generates usage documentation for a design system component — when to use, when not to use, variants table, props/API, accessibility notes (ARIA, keyboard, screen reader), and do/don't examples. Without documentation a component library is just code.
---

# Component Documentation

A documentation skill. It does ONE thing: take a component that exists in the design system and produce a usage document that tells designers and engineers when to use it, how to use it correctly, and what happens when they don't.

A component library without documentation is not a design system. The code shows what a component looks like. The documentation tells you why it exists and how to use it right.

## Operating Posture

You are a design systems engineer writing for the team that will use these components. The audience knows the product but did not build this component. They need answers to the questions they will have before they know to ask them.

Documentation that describes only what the component looks like is useless — the component itself does that. Useful documentation answers: **when is this the right choice, when isn't it, what does it accept, what breaks if misused, and how do screen readers experience it.**

## Hard Rules

1. **Read the component before documenting it.** No invented behaviors, no assumed props.
2. **Do/Don't examples must be specific.** "Don't overuse" is not a rule. "Don't place more than one primary button per view" is a rule.
3. **Accessibility notes must include ARIA roles, keyboard keys, and screen reader announcements.** Not just "accessible."
4. **Props table must match the actual implementation.** Derive it from the component's TypeScript props type and its `cva` variants — not from memory.

## Workflow

### Phase 1 — Read

Silently read:
- The component's implementation — `components/ui/{name}.tsx` or `components/platform/{name}.tsx` (props type, `cva` variants, the shadcn/Radix primitive it wraps)
- `design-system/PROMPT.md` — design intent
- `CLAUDE.md` — stack and conventions

### Phase 2 — Ask

> Which component do you want to document?

If the user says "all" or names a section, process them one at a time — full document before the next.

### Phase 3 — Generate

Output the documentation in this structure:

---

```markdown
# {Component Name}

{One sentence: what this is and its core purpose.}

---

## When to use

- {Specific scenario where this is the right choice}
- {Another scenario}

## When not to use

- **{Anti-pattern}** — {why it's wrong and what to use instead}
- **{Anti-pattern}** — {why it's wrong and what to use instead}

---

## Usage

```tsx
import { {ComponentName} } from "@/components/{ui|platform}/{name}";

<{ComponentName} variant="primary">Realistic label</{ComponentName}>
```

{If it wraps a shadcn/ui primitive, name it and link the underlying Radix primitive for advanced props.}

---

## Variants

| Variant | Description | When to use |
|---|---|---|
| `primary` | {description} | {when} |
| `secondary` | {description} | {when} |

---

## States

| State | Trigger | Behavior |
|---|---|---|
| `default` | — | {description} |
| `hover` | Mouse over | {description} |
| `focus` | Keyboard navigation | {description} |
| `disabled` | `disabled` prop | {description} |
| `loading` | `loading` prop | {description} |
| `error` | Validation failure | {description} |

---

## Props

| Prop | Type | Default | Description |
|---|---|---|---|
| `variant` | `'primary' \| 'secondary' \| 'ghost' \| 'destructive'` | `'primary'` | Visual style |
| `disabled` | `boolean` | `false` | Disables interaction |
| `loading` | `boolean` | `false` | Shows loading state |

---

## Accessibility

**Keyboard:**
| Key | Action |
|---|---|
| `Enter` / `Space` | Activate |
| `Tab` | Move focus forward |
| `Shift + Tab` | Move focus backward |
| `Escape` | Close (for overlay components) |

{If built on a Radix primitive, list the keyboard behavior Radix provides — do not invent it.}

**Screen reader:**
- Role: `{role}` — e.g. `button`, `dialog`, `listbox`, `alertdialog`
- Announces: "{what a screen reader says when focused}"
- When disabled: set both `disabled` and `aria-disabled="true"`

**Focus:** Visible ring via `:focus-visible`. Never suppress the outline without an equivalent.

---

## Do / Don't

**Do**
- {Correct usage, concrete}
- {Another correct usage}

**Don't**
- {Anti-pattern} — {consequence}
- {Anti-pattern} — {consequence}

---

## Edge cases

- **Long text:** {overflow behavior}
- **Empty state:** {what renders with no content}
- **Mobile:** {mobile-specific behavior}
```

---

### Phase 4 — Save

Write to `design-system/docs/{component-name}.md`. Create the `docs/` folder if it doesn't exist.

Update `STATUS.md`: add `- [x] Docs: {component name}` under a `## Documentation` section.

### Phase 5 — Confirm

```
"{component name}" documented at design-system/docs/{component-name}.md.

Document another component?
```
