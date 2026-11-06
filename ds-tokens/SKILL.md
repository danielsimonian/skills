---
name: ds-tokens
description: Designs the full design token architecture — primitive layer, semantic layer, dark/light mode, and motion tokens (Emil Kowalski's easing curves baked in). Produces a single CSS custom properties file that becomes the token source of truth for the entire design system.
---

# Design Token Architecture

A foundation skill. It does ONE thing: take a product brief and visual direction, then produce a complete, two-layer token system — primitives and semantics — with dark mode and motion tokens built in from the start.

This is the work that must happen before building components. A design system without a deliberate token architecture is a system that fights you every time you change a color, add a theme, or try to make components consistent.

## Operating Posture

You are a design systems engineer laying the foundation. The goal is not to pick colors — it is to design a **naming system** that scales: one where any designer or engineer can find the right token without guessing, and where switching themes or rebranding requires touching only the primitive layer.

A token system that conflates primitives with semantics (`--blue-500` used in a button) is a system that will resist change. A well-designed system expresses intent at the semantic layer (`--color-action-primary`) and resolves it at the primitive layer.

## Hard Rules

1. **Two mandatory layers.** Primitives and semantics. Components reference only semantic tokens, never primitives directly.
2. **Semantics express intent, not appearance.** `--color-action-primary` is correct. `--color-blue` in a button is not.
3. **Dark mode lives in the semantic layer only.** Primitives are brand values that don't change. Only semantics remap.
4. **Motion tokens are not optional.** Emil's easing curves and duration scale belong in the token system.
5. **Every token needs a reason.** If two tokens always resolve to the same primitive and serve the same intent, merge them.

## Workflow

### Phase 1 — Read the brief

Read `CLAUDE.md` and `design-system/PROMPT.md` silently. Extract the visual tone, brand colors, and style direction.

If no project files exist, ask:
> To design the token system, I need:
> 1. **Brand color** — hex, name, or tone description
> 2. **Visual tone** — e.g. "minimal dark", "vibrant consumer", "neutral enterprise"
> 3. **Multiple themes needed?** (beyond dark/light)

### Phase 2 — Primitive layer

Raw values. No semantic meaning. Never referenced directly in components.

```css
/* ─── PRIMITIVES ─── raw values only, never used in components ─── */
:root {
  /* Brand scale (derive from product's hue) */
  --p-brand-50:  oklch(97% 0.02 265);
  --p-brand-100: oklch(93% 0.05 265);
  --p-brand-200: oklch(87% 0.08 265);
  --p-brand-300: oklch(78% 0.12 265);
  --p-brand-400: oklch(68% 0.16 265);
  --p-brand-500: oklch(58% 0.20 265);
  --p-brand-600: oklch(48% 0.20 265);
  --p-brand-700: oklch(40% 0.18 265);
  --p-brand-800: oklch(32% 0.15 265);
  --p-brand-900: oklch(24% 0.12 265);
  --p-brand-950: oklch(16% 0.08 265);

  /* Neutral scale (slightly tinted toward brand hue) */
  --p-neutral-0:    oklch(100% 0 0);
  --p-neutral-50:   oklch(98% 0.002 265);
  --p-neutral-100:  oklch(95% 0.004 265);
  --p-neutral-200:  oklch(90% 0.006 265);
  --p-neutral-300:  oklch(82% 0.008 265);
  --p-neutral-400:  oklch(70% 0.008 265);
  --p-neutral-500:  oklch(58% 0.008 265);
  --p-neutral-600:  oklch(46% 0.008 265);
  --p-neutral-700:  oklch(36% 0.008 265);
  --p-neutral-800:  oklch(26% 0.008 265);
  --p-neutral-900:  oklch(18% 0.008 265);
  --p-neutral-950:  oklch(12% 0.006 265);
  --p-neutral-1000: oklch(7%  0.004 265);

  /* Feedback */
  --p-error-400:   oklch(62% 0.22 25);
  --p-error-500:   oklch(55% 0.22 25);
  --p-error-600:   oklch(47% 0.20 25);
  --p-error-50:    oklch(97% 0.04 25);
  --p-warning-500: oklch(72% 0.18 75);
  --p-warning-50:  oklch(98% 0.04 75);
  --p-success-500: oklch(65% 0.18 145);
  --p-success-50:  oklch(97% 0.04 145);
  --p-info-500:    oklch(60% 0.18 240);
  --p-info-50:     oklch(97% 0.04 240);

  /* Motion durations */
  --p-dur-instant: 100ms;
  --p-dur-fast:    160ms;
  --p-dur-normal:  250ms;
  --p-dur-slow:    400ms;
  --p-dur-drawer:  500ms;
}
```

### Phase 3 — Semantic layer (light mode default)

```css
/* ─── SEMANTICS ─── intent-based aliases, used in all components ─── */
:root {
  /* Backgrounds */
  --color-bg-base:    var(--p-neutral-50);
  --color-bg-raised:  var(--p-neutral-0);
  --color-bg-overlay: var(--p-neutral-100);
  --color-bg-sunken:  var(--p-neutral-100);

  /* Surfaces */
  --color-surface-default: var(--p-neutral-0);
  --color-surface-hover:   var(--p-neutral-50);
  --color-surface-active:  var(--p-neutral-100);

  /* Text */
  --color-text-primary:   var(--p-neutral-950);
  --color-text-secondary: var(--p-neutral-600);
  --color-text-placeholder: var(--p-neutral-400);
  --color-text-disabled:  var(--p-neutral-400);
  --color-text-inverse:   var(--p-neutral-0);
  --color-text-on-action: var(--p-neutral-0);

  /* Borders */
  --color-border-default: var(--p-neutral-200);
  --color-border-strong:  var(--p-neutral-400);
  --color-border-focus:   var(--p-brand-500);

  /* Actions */
  --color-action-primary:         var(--p-brand-600);
  --color-action-primary-hover:   var(--p-brand-700);
  --color-action-primary-active:  var(--p-brand-800);
  --color-action-secondary:       var(--p-neutral-100);
  --color-action-secondary-hover: var(--p-neutral-200);
  --color-action-ghost-hover:     var(--p-neutral-100);
  --color-action-destructive:       var(--p-error-500);
  --color-action-destructive-hover: var(--p-error-600);

  /* Feedback */
  --color-feedback-error:      var(--p-error-500);
  --color-feedback-error-bg:   var(--p-error-50);
  --color-feedback-warning:    var(--p-warning-500);
  --color-feedback-warning-bg: var(--p-warning-50);
  --color-feedback-success:    var(--p-success-500);
  --color-feedback-success-bg: var(--p-success-50);
  --color-feedback-info:       var(--p-info-500);
  --color-feedback-info-bg:    var(--p-info-50);

  /* Typography */
  --font-sans: 'Inter', system-ui, sans-serif;
  --font-mono: 'JetBrains Mono', 'Fira Code', monospace;
  --font-size-xs:  0.75rem;
  --font-size-sm:  0.875rem;
  --font-size-md:  1rem;
  --font-size-lg:  1.125rem;
  --font-size-xl:  1.25rem;
  --font-size-2xl: 1.5rem;
  --font-size-3xl: 1.875rem;
  --font-size-4xl: 2.25rem;
  --font-weight-regular:  400;
  --font-weight-medium:   500;
  --font-weight-semibold: 600;
  --font-weight-bold:     700;
  --line-height-tight:   1.25;
  --line-height-normal:  1.5;
  --line-height-relaxed: 1.75;

  /* Spacing (4px base) */
  --space-1: 0.25rem;  --space-2: 0.5rem;   --space-3: 0.75rem;
  --space-4: 1rem;     --space-5: 1.25rem;  --space-6: 1.5rem;
  --space-8: 2rem;     --space-10: 2.5rem;  --space-12: 3rem;
  --space-16: 4rem;    --space-20: 5rem;    --space-24: 6rem;

  /* Border radius */
  --radius-sm:   4px;
  --radius-md:   8px;
  --radius-lg:   12px;
  --radius-xl:   16px;
  --radius-2xl:  24px;
  --radius-full: 9999px;

  /* Shadows */
  --shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.08), 0 2px 4px -2px rgb(0 0 0 / 0.05);
  --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.08), 0 4px 6px -4px rgb(0 0 0 / 0.05);
  --shadow-xl: 0 20px 25px -5px rgb(0 0 0 / 0.08), 0 8px 10px -6px rgb(0 0 0 / 0.05);

  /* Motion — Emil Kowalski */
  --ease-out:    cubic-bezier(0.23, 1, 0.32, 1);
  --ease-in-out: cubic-bezier(0.77, 0, 0.175, 1);
  --ease-drawer: cubic-bezier(0.32, 0.72, 0, 1);
  --dur-instant: var(--p-dur-instant);
  --dur-fast:    var(--p-dur-fast);
  --dur-normal:  var(--p-dur-normal);
  --dur-slow:    var(--p-dur-slow);
  --dur-drawer:  var(--p-dur-drawer);

  /* Z-index */
  --z-raised:   10;
  --z-dropdown: 100;
  --z-sticky:   200;
  --z-overlay:  300;
  --z-modal:    400;
  --z-toast:    500;
  --z-tooltip:  600;
}
```

### Phase 4 — Dark mode

Only the semantic layer remaps. Primitives stay fixed.

```css
@media (prefers-color-scheme: dark) {
  :root {
    --color-bg-base:    var(--p-neutral-950);
    --color-bg-raised:  var(--p-neutral-900);
    --color-bg-overlay: var(--p-neutral-800);
    --color-bg-sunken:  var(--p-neutral-1000);

    --color-surface-default: var(--p-neutral-900);
    --color-surface-hover:   var(--p-neutral-800);
    --color-surface-active:  var(--p-neutral-700);

    --color-text-primary:     var(--p-neutral-50);
    --color-text-secondary:   var(--p-neutral-400);
    --color-text-placeholder: var(--p-neutral-600);
    --color-text-disabled:    var(--p-neutral-600);

    --color-border-default: var(--p-neutral-800);
    --color-border-strong:  var(--p-neutral-600);

    --color-action-primary:         var(--p-brand-400);
    --color-action-primary-hover:   var(--p-brand-300);
    --color-action-secondary:       var(--p-neutral-800);
    --color-action-secondary-hover: var(--p-neutral-700);
    --color-action-ghost-hover:     var(--p-neutral-800);

    --shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.3);
    --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.4), 0 2px 4px -2px rgb(0 0 0 / 0.3);
    --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.4), 0 4px 6px -4px rgb(0 0 0 / 0.3);
    --shadow-xl: 0 20px 25px -5px rgb(0 0 0 / 0.4), 0 8px 10px -6px rgb(0 0 0 / 0.3);
  }
}

[data-theme="dark"]  { /* same as above */ }
[data-theme="light"] { /* explicit light values for override */ }
```

### Phase 5 — Save and advise

Write the token file to `design-system/tokens.css`. Update `STATUS.md`.

```
Token system saved to design-system/tokens.css.

{n} tokens across 2 layers:
  Primitive: brand scale, neutral scale, feedback, motion durations
  Semantic:  backgrounds, surfaces, text, borders, actions, feedback,
             typography, spacing, radii, shadows, motion, z-index

Dark mode: semantic remap via prefers-color-scheme + [data-theme].

Rule: components import tokens.css and reference only semantic tokens.
Never --p-* directly in a component.
```
