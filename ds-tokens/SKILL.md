---
name: ds-tokens
description: Designs the full design token architecture — primitive layer, semantic layer, dark/light mode, and motion tokens (Emil Kowalski's easing curves baked in). Writes them into the Next.js app's `app/globals.css` as Tailwind v4 theme tokens, bridged to shadcn/ui, with dark mode via next-themes — the single token source of truth for the entire design system.
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
6. **Next.js + Tailwind v4 + shadcn/ui.** Tokens live in `app/globals.css`, semantics are Tailwind theme variables, and shadcn/ui's variables are bridged to them. Never a separate standalone CSS file.

## Workflow

### Phase 1 — Read the brief

Read `CLAUDE.md`, `design-system/PROMPT.md`, and the current `app/globals.css` silently. Extract the visual tone, brand colors, and style direction. If there is no Next.js app (`package.json` + `app/`), stop and tell the user to run `/designsystem` first.

If it was invoked from `/designsystem`, do not ask anything — the brief is already in the project files. If it was run on its own and no project files exist, ask:
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

### Phase 3 — Semantic layer (Tailwind v4 `@theme`, light values)

Semantics go inside `@theme` so Tailwind generates utilities from them (`bg-bg-base`, `text-text-primary`, `p-4`, `rounded-md`, `ease-out`). Names follow Tailwind's namespaces: `--color-*`, `--spacing-*`, `--text-*`, `--font-*`, `--font-weight-*`, `--leading-*`, `--radius-*`, `--shadow-*`, `--ease-*`.

`--color-*: initial;` removes Tailwind's default palette, so `bg-blue-500` stops existing and only system colors can be used.

Use plain `@theme` (not `@theme inline`) for anything that changes in dark mode — utilities then reference `var(--color-…)`, and the `.dark` overrides in Phase 4 take effect.

```css
/* ─── SEMANTICS ─── intent-based, the only tokens components may use ─── */
@theme {
  --color-*: initial;

  /* Kept because shadcn/ui overlays use bg-black/50 */
  --color-black: var(--p-neutral-1000);
  --color-white: var(--p-neutral-0);

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
  --color-text-primary:     var(--p-neutral-950);
  --color-text-secondary:   var(--p-neutral-600);
  --color-text-placeholder: var(--p-neutral-400);
  --color-text-disabled:    var(--p-neutral-400);
  --color-text-inverse:     var(--p-neutral-0);
  --color-text-on-action:   var(--p-neutral-0);

  /* Borders */
  --color-border-default: var(--p-neutral-200);
  --color-border-strong:  var(--p-neutral-400);
  --color-border-focus:   var(--p-brand-500);

  /* Actions */
  --color-action-primary:           var(--p-brand-600);
  --color-action-primary-hover:     var(--p-brand-700);
  --color-action-primary-active:    var(--p-brand-800);
  --color-action-secondary:         var(--p-neutral-100);
  --color-action-secondary-hover:   var(--p-neutral-200);
  --color-action-ghost-hover:       var(--p-neutral-100);
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

  /* Typography — fonts loaded with next/font, exposed as CSS variables */
  --font-sans: var(--font-inter), system-ui, sans-serif;
  --font-mono: var(--font-jetbrains-mono), ui-monospace, monospace;
  --text-xs:  0.75rem;
  --text-sm:  0.875rem;
  --text-md:  1rem;
  --text-lg:  1.125rem;
  --text-xl:  1.25rem;
  --text-2xl: 1.5rem;
  --text-3xl: 1.875rem;
  --text-4xl: 2.25rem;
  --font-weight-regular:  400;
  --font-weight-medium:   500;
  --font-weight-semibold: 600;
  --font-weight-bold:     700;
  --leading-tight:   1.25;
  --leading-normal:  1.5;
  --leading-relaxed: 1.75;

  /* Spacing (4px base) */
  --spacing-1: 0.25rem;  --spacing-2: 0.5rem;   --spacing-3: 0.75rem;
  --spacing-4: 1rem;     --spacing-5: 1.25rem;  --spacing-6: 1.5rem;
  --spacing-8: 2rem;     --spacing-10: 2.5rem;  --spacing-12: 3rem;
  --spacing-16: 4rem;    --spacing-20: 5rem;    --spacing-24: 6rem;

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

  /* Easing — Emil Kowalski (utilities: ease-out, ease-in-out, ease-drawer) */
  --ease-out:    cubic-bezier(0.23, 1, 0.32, 1);
  --ease-in-out: cubic-bezier(0.77, 0, 0.175, 1);
  --ease-drawer: cubic-bezier(0.32, 0.72, 0, 1);
}

/* Tokens without a Tailwind namespace — use as duration-(--dur-fast), z-(--z-modal) */
:root {
  --dur-instant: var(--p-dur-instant);
  --dur-fast:    var(--p-dur-fast);
  --dur-normal:  var(--p-dur-normal);
  --dur-slow:    var(--p-dur-slow);
  --dur-drawer:  var(--p-dur-drawer);

  --z-raised:   10;
  --z-dropdown: 100;
  --z-sticky:   200;
  --z-overlay:  300;
  --z-modal:    400;
  --z-toast:    500;
  --z-tooltip:  600;
}
```

### Phase 4 — Dark mode (`.dark` class via next-themes)

Only the semantic layer remaps. Primitives stay fixed. Dark is the default theme.

```css
@custom-variant dark (&:where(.dark, .dark *));

.dark {
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
```

Wire the theme in `app/layout.tsx`:

```tsx
import { ThemeProvider } from "next-themes";
import { Inter, JetBrains_Mono } from "next/font/google";

const inter = Inter({ subsets: ["latin"], variable: "--font-inter" });
const mono = JetBrains_Mono({ subsets: ["latin"], variable: "--font-jetbrains-mono" });

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" suppressHydrationWarning className={`${inter.variable} ${mono.variable}`}>
      <body className="bg-bg-base text-text-primary font-sans antialiased">
        <ThemeProvider attribute="class" defaultTheme="dark" enableSystem>
          {children}
        </ThemeProvider>
      </body>
    </html>
  );
}
```

Swap the fonts for the ones the visual direction calls for — always through `next/font`.

### Phase 5 — Bridge shadcn/ui to the semantic layer

shadcn/ui components read their own variables (`--background`, `--primary`, `--ring`…). Point them at the semantic tokens so every installed primitive picks up the system automatically — and replace the color block that `shadcn init` generated in `globals.css`:

```css
:root {
  --background:             var(--color-bg-base);
  --foreground:             var(--color-text-primary);
  --card:                   var(--color-bg-raised);
  --card-foreground:        var(--color-text-primary);
  --popover:                var(--color-bg-overlay);
  --popover-foreground:     var(--color-text-primary);
  --primary:                var(--color-action-primary);
  --primary-foreground:     var(--color-text-on-action);
  --secondary:              var(--color-action-secondary);
  --secondary-foreground:   var(--color-text-primary);
  --muted:                  var(--color-bg-sunken);
  --muted-foreground:       var(--color-text-secondary);
  --accent:                 var(--color-action-ghost-hover);
  --accent-foreground:      var(--color-text-primary);
  --destructive:            var(--color-action-destructive);
  --border:                 var(--color-border-default);
  --input:                  var(--color-border-default);
  --ring:                   var(--color-border-focus);
  --radius:                 var(--radius-md);
}

@theme inline {
  --color-background:           var(--background);
  --color-foreground:           var(--foreground);
  --color-card:                 var(--card);
  --color-card-foreground:      var(--card-foreground);
  --color-popover:              var(--popover);
  --color-popover-foreground:   var(--popover-foreground);
  --color-primary:              var(--primary);
  --color-primary-foreground:   var(--primary-foreground);
  --color-secondary:            var(--secondary);
  --color-secondary-foreground: var(--secondary-foreground);
  --color-muted:                var(--muted);
  --color-muted-foreground:     var(--muted-foreground);
  --color-accent:               var(--accent);
  --color-accent-foreground:    var(--accent-foreground);
  --color-destructive:          var(--destructive);
  --color-border:               var(--border);
  --color-input:                var(--input);
  --color-ring:                 var(--ring);
}
```

No `.dark` block is needed for the bridge — the shadcn variables resolve through the semantic tokens, which already remap.

If the project is on Tailwind v3 (`tailwind.config.ts` exists), keep the same `:root` / `.dark` variables and map them in `theme.extend` with `var(--…)` instead of `@theme`, with `darkMode: "class"`.

### Phase 6 — Save and advise

Write everything to `app/globals.css` in this order: `@import "tailwindcss"` → dark variant → primitives (`:root`) → semantics (`@theme`) → non-namespaced tokens (`:root`) → dark overrides (`.dark`) → shadcn bridge. Update `app/layout.tsx` as in Phase 4. Run `npm run build` to confirm it compiles. Update `STATUS.md`.

```
Token system saved to app/globals.css.

{n} tokens across 2 layers:
  Primitive: brand scale, neutral scale, feedback, motion durations
  Semantic:  backgrounds, surfaces, text, borders, actions, feedback,
             typography, spacing, radii, shadows, motion, z-index

Dark mode (default): .dark class via next-themes — semantic remap only.
shadcn/ui: bridged — every installed primitive uses the system tokens.

Rule: components use only token-backed Tailwind classes
(bg-action-primary, text-text-secondary, rounded-md, ease-out).
Never --p-*, never arbitrary values, never the default palette.
```
