---
name: ds-audit
description: Audits existing design system components for consistency issues — token violations, missing states, incorrect motion, accessibility gaps. Outputs a prioritized Before/After table in Emil Kowalski's review format. Run after building components to close the quality loop.
---

# Design System Audit

A quality-gate skill. It does ONE thing: review components already built in the design system and produce a prioritized list of violations in Emil Kowalski's Before/After table format — specific, actionable, no fluff.

## Operating Posture

You are a senior design engineer doing a strict review. The value of this skill is **precision**: a vague "looks inconsistent" is useless. Every finding must name the exact property and expected value. The output mirrors Emil's review format — a markdown table with Before, After, and Why columns.

## Hard Rules

1. **Read the system before reviewing.** Understand the established tokens, states, and motion rules before judging a component against them.
2. **Only flag real violations.** An intentional deviation for a valid reason is not a violation. If unclear, note it as a warning, not an error.
3. **Prioritize by impact.** Accessibility first, then token violations, then missing states, then motion, then cosmetic.
4. **Every finding must be actionable.** If it cannot be fixed with a specific code change, it is not a finding.

## Workflow

### Phase 1 — Read the system

Read silently before anything else:
- `CLAUDE.md` — stack, conventions, motion rules
- `design-system/PROMPT.md` — visual language and component inventory
- `STATUS.md` — which components are marked done

### Phase 2 — Identify what to audit

Ask in a single message:

> What do you want to audit?
>
> 1. **A specific component** — e.g. "Button", "Toast", "Modal"
> 2. **A whole section** — "Foundations", "Core Components", or "Platform Components"
> 3. **Everything** — full audit of all components marked done in STATUS.md

### Phase 3 — Run the audit

For each component in scope, check every category below.

#### A — Accessibility (highest priority)
- [ ] Interactive elements have correct `role`
- [ ] `aria-label` or `aria-labelledby` where needed
- [ ] `aria-disabled` alongside HTML `disabled`
- [ ] Visible focus ring using `:focus-visible` (not `:focus`)
- [ ] No positive `tabindex` values
- [ ] WCAG AA contrast (4.5:1 text, 3:1 UI elements)
- [ ] Hover states behind `@media (hover: hover) and (pointer: fine)`

#### B — Tokens (no hardcoded values)
- [ ] No hardcoded colors — only `var(--color-*)`
- [ ] No hardcoded font sizes — only `var(--font-*)`
- [ ] No hardcoded spacing — only `var(--spacing-*)`
- [ ] No hardcoded radii — only `var(--radius-*)`
- [ ] No hardcoded shadows — only `var(--shadow-*)`
- [ ] Easing: only `var(--ease-out)`, `var(--ease-in-out)`, `var(--ease-drawer)`

#### C — States
- [ ] `default` present and correct
- [ ] `hover` present and visually distinct
- [ ] `focus-visible` present (not `:focus` alone)
- [ ] `disabled`: reduced opacity + `cursor: not-allowed` + `aria-disabled`
- [ ] `loading` where applicable
- [ ] `error` where applicable
- [ ] `selected`/`active` where applicable

#### D — Motion (Emil's rules)
- [ ] No animation on keyboard-triggered actions
- [ ] Duration: buttons 100–160ms, tooltips 125–200ms, modals/drawers 200–500ms
- [ ] Easing: ease-out for entrances, ease-in-out for on-screen movement, never ease-in
- [ ] Never from `scale(0)` — always `scale(0.95) + opacity: 0`
- [ ] Popovers/dropdowns: `transform-origin` from trigger (modals: center)
- [ ] CSS transitions (not keyframes) for interruptible UI
- [ ] `prefers-reduced-motion` applied
- [ ] Only `transform` and `opacity` animated

#### E — Composition and responsiveness
- [ ] Larger components use existing pieces (no duplicated markup)
- [ ] No hardcoded inline CSS inside composed components
- [ ] Works at 320px without horizontal overflow

### Phase 4 — Output the findings

Format every violation as a Before/After table per component, ordered A → E:

```markdown
## Audit — {component name}

| Category | Before | After | Why |
|---|---|---|---|
| Accessibility | `<button disabled>` | `<button disabled aria-disabled="true">` | `disabled` alone isn't announced by all screen readers |
| Token | `color: #1a1a2e` | `color: var(--color-text-primary)` | Hardcoded value breaks theming and dark mode |
| Motion | `transition: all 200ms ease-in` | `transition: transform 160ms var(--ease-out)` | `all` animates unnecessary properties; `ease-in` starts slow and feels sluggish |
| State | No `focus-visible` | `outline: 2px solid var(--color-border-focus)` on `:focus-visible` | Keyboard users have no visual focus indication |
```

If no violations: `{component name}: no violations found.`

### Phase 5 — Summary

```
AUDIT SUMMARY
{n} components reviewed — {n} violations found

Immediate (accessibility): {list}
High priority (tokens, states): {list}
Normal (motion, composition): {list}
```

Offer to fix violations starting with the highest priority.
