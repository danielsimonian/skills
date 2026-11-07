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
- `app/globals.css` — the token source of truth (primitives, `@theme` semantics, `.dark` overrides, shadcn bridge)
- The component files in `components/ui/` and `components/platform/`, plus their catalog entries in `app/page.tsx`

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
- [ ] Hover in custom CSS behind `@media (hover: hover) and (pointer: fine)` (Tailwind v4 `hover:` already does this)
- [ ] Radix/shadcn accessibility preserved — no `asChild` misuse, no removed `aria-*` or focus handling from the primitive

#### B — Tokens (no hardcoded values)
- [ ] No arbitrary values in `className` — no `bg-[#1a1a2e]`, `p-[13px]`, `rounded-[10px]`, `text-[15px]`
- [ ] No hex/rgb/px values in `style={{}}` props
- [ ] Colors only from semantic utilities (`bg-action-primary`, `text-text-secondary`, `border-border-default`) or the shadcn bridge (`bg-primary`, `ring-ring`)
- [ ] No primitives (`--p-*`) referenced in components
- [ ] Spacing, radii, shadows, font sizes from the theme scale (`p-4`, `rounded-md`, `shadow-lg`, `text-sm`)
- [ ] Easing only `ease-out`, `ease-in-out`, `ease-drawer`; durations via `duration-(--dur-*)`
- [ ] New tokens added to `app/globals.css`, not invented inline

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
- [ ] Larger components import existing pieces from `components/ui/` and `components/platform/` (no duplicated markup)
- [ ] shadcn/ui primitives used where they exist — not rebuilt from scratch
- [ ] Works at 320px without horizontal overflow

#### F — Next.js / React
- [ ] TypeScript strict: props typed, no `any`
- [ ] Variants defined with `cva`, conditional classes with `cn()` — no string concatenation
- [ ] `"use client"` only on components that need state, effects, or event handlers
- [ ] Images via `next/image`, fonts via `next/font`
- [ ] Functional components + hooks only; shared state in Zustand, no prop drilling
- [ ] `npm run build` and `npm run lint` pass

### Phase 4 — Output the findings

Format every violation as a Before/After table per component, ordered A → F:

```markdown
## Audit — {component name}

| Category | Before | After | Why |
|---|---|---|---|
| Accessibility | `<Button disabled>` | `<Button disabled aria-disabled="true">` | `disabled` alone isn't announced by all screen readers |
| Token | `className="text-[#1a1a2e]"` | `className="text-text-primary"` | Arbitrary value breaks theming and dark mode |
| Motion | `transition-all duration-200 ease-in` | `transition-transform duration-(--dur-fast) ease-out` | `all` animates unnecessary properties; `ease-in` starts slow and feels sluggish |
| State | No focus style | `focus-visible:ring-2 focus-visible:ring-ring` | Keyboard users have no visual focus indication |
| Next.js | `"use client"` on a static card | Remove the directive | Ships unnecessary JS to the client |
```

If no violations: `{component name}: no violations found.`

### Phase 5 — Summary

```
AUDIT SUMMARY
{n} components reviewed — {n} violations found

Immediate (accessibility): {list}
High priority (tokens, states): {list}
Normal (motion, composition, Next.js): {list}
```

Offer to fix violations starting with the highest priority.
