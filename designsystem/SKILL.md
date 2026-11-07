---
name: designsystem
description: Scaffolds a new Next.js (App Router + Tailwind + shadcn/ui) design system project from scratch — creates the folder structure, generates a tailored component brief, a visual foundations spec, and a progress tracker. Embeds Emil Kowalski's design engineering philosophy (animation decisions, component polish, performance rules) into every generated document so all future sessions apply them automatically. Only runs when explicitly invoked; it does not trigger on its own.
---

# Design System Bootstrap

A setup skill. It does ONE thing: take a product brief and visual references, then produce the scaffolding and specification documents that make a design system project coherent from day one — a living component brief, a progress tracker, and a clear foundation to build from.

**Motion and polish rules come from Emil Kowalski's `emil-design-eng` skill.** They are embedded in every document this skill generates, so components and screens built later automatically apply the same craft bar without loading the skill separately.

## Operating Posture

You are a senior design engineer setting up a new design system. The goal of this skill is not to build anything yet — it is to create the right **foundation**: a brief that is specific enough to make future decisions obvious, a component inventory that reflects what this product actually needs (not a generic checklist), and a structure that another agent (or you, next session) can pick up and continue without losing context.

A good brief from this skill should answer, without ambiguity: what does this product do, who uses it, what is the visual language, and which components are needed first.

## Hard Rules

1. **Run the whole flow without stopping.** After the brief is collected (Phase 1), execute every phase through to the end — scaffold, documents, tokens, catalog shell, build check — with no confirmation questions, no "shall I continue?", no waiting for files. Make reasonable decisions on your own and list them in the final summary; the user reviews and changes things once everything is ready. The only valid reasons to stop are a missing project name/brief or a command that fails.
2. **The component inventory must be product-specific.** Do not copy-paste a generic list. Derive platform-specific components from the brief — an AI chat product needs a prompt field and message bubbles; a fintech needs transaction cards and balance meters.
3. **Visual language comes from references when they exist, from the brief when they don't.** Use links, images and the tone described in the brief. If there are no references, infer a coherent direction from the brief and record it as an assumption in the final summary — never stop to ask.
4. **Status tracking from the start.** Generate a `STATUS.md` checklist at setup time. Future sessions use it to know what is done and what is next.
5. **The stack is always Next.js.** Every design system is built in Next.js (App Router) + TypeScript + Tailwind CSS + shadcn/ui. Do not ask the user to choose a stack and do not fall back to plain HTML/CSS/JS.
6. **The catalog always has the same shell.** Every project ships with the catalog built in Phase 7: a sticky sidebar with grouped navigation (a drawer on mobile), a header with the project name, version and a dark/light theme toggle, and anchored sections. This is not optional and not left for "later" — the user must be able to open `npm run dev`, switch themes and navigate the whole system on day one.

## Workflow

### Phase 1 — Collect the brief

If the user already gave a project name and brief together with the command (e.g. `/designsystem Orbit — AI chat app for…`), skip this phase entirely and start Phase 2a.

Otherwise, ask **once**, in a single message — this is the only question the skill asks:

> To set up the design system, send me in one message:
>
> 1. **Project name**
> 2. **Product brief** — 2–5 sentences: what it is, who uses it, the visual tone you have in mind
> 3. **References** (optional) — links (Figma, Dribbble, sites) and/or the path of a folder with images
>
> After that I'll run the whole flow without stopping — you review and adjust at the end.

Wait for this one reply. Missing references are fine; never ask a follow-up.

### Phase 2a — Scaffold the Next.js app and folder structure

From `~/Documents/Claude/Projects/`, create the Next.js app first (it must run in an empty folder). Use the project name in kebab-case for the folder — npm rejects uppercase and spaces:

```bash
npx create-next-app@latest {project-name} --ts --tailwind --eslint --app --no-src-dir --import-alias "@/*" --use-npm --yes
cd {project-name}
npx shadcn@latest init -d
npm install next-themes zustand
```

Then add the design system files, so the project ends up as:

```
{project-name}/
├── CLAUDE.md
├── STATUS.md
├── app/
│   ├── globals.css        ← token source of truth (filled by /ds-tokens)
│   ├── layout.tsx
│   └── page.tsx           ← catalog page (built in Phase 7)
├── components/
│   ├── ui/                ← shadcn/ui primitives, restyled with tokens
│   ├── platform/          ← product-specific components
│   └── catalog/           ← catalog shell: sidebar, header, theme toggle, sections
├── lib/utils.ts           ← cn()
└── design-system/
    ├── PROMPT.md
    ├── docs/
    └── references/
        └── .gitkeep
```

If `create-next-app` generated its own `CLAUDE.md` or `AGENTS.md`, merge its content into the `CLAUDE.md` written in Phase 3 instead of keeping two files. If any scaffolding command fails, report the error and stop — do not fall back to a non-Next structure.

### Phase 2b — Gather visual references (no waiting)

- If the user gave a folder of images, copy them into `design-system/references/` and look at them.
- If they gave links, fetch them when possible and extract colors, typography, density and tone.
- If there are no references, derive the direction from the brief.

Continue straight to Phase 3. Do not ask the user to add files and do not wait.

### Phase 3 — Write CLAUDE.md

A concise context file for future sessions. Include:

- Project name and brief (1–3 sentences)
- Stack (always): Next.js (App Router) + TypeScript strict + Tailwind CSS + shadcn/ui
  - Tokens live in `app/globals.css`: primitives on `:root`, semantics in Tailwind v4 `@theme`, dark overrides in `.dark` (via `next-themes`, `defaultTheme="dark"`)
  - Components use only token-backed Tailwind classes — no arbitrary values (`bg-[#...]`, `p-[13px]`), no default palette (`bg-blue-500`)
  - shadcn/ui for primitives — restyle them with the system's tokens, never rebuild them from scratch
  - Components in `components/ui/` (primitives) and `components/platform/` (product-specific)
  - Catalog page at `app/page.tsx`, inside the shell in `components/catalog/` (sidebar + header with theme toggle). Navigation lives in `components/catalog/nav.ts` — every new component or screen gets an entry there
  - `cn()` for conditional classes, `next/image` for images, functional components + hooks only
- Design system conventions:
  - Dark mode as default
  - CSS custom properties for all tokens (`--color-*`, `--spacing-*`, `--radius-*`, `--font-*`, `--ease-*`)
  - Components must be reusable and composable — never single-use
  - Required states: default, hover, focus-visible, disabled; add loading and error where applicable
  - Realistic content in examples, no placeholder text
  - WCAG AA contrast minimum
  - Motion rules (from `emil-design-eng`): never animate keyboard actions; UI cap 300ms; ease-out for entrances; only `transform`/`opacity`; `prefers-reduced-motion` required; easing tokens: `--ease-out: cubic-bezier(0.23,1,0.32,1)`, `--ease-in-out: cubic-bezier(0.77,0,0.175,1)`, `--ease-drawer: cubic-bezier(0.32,0.72,0,1)`
- Visual references list (links or folder path)

### Phase 4 — Write STATUS.md

A flat checklist the user will update across sessions. Structure:

```markdown
# {project-name} — Design System Status

**Created:** {date}
**Status:** 🔵 In progress

---

## Visual Foundations
- [ ] Color palette (background, surface, text, border, action, feedback)
- [ ] Typography (headings, body, captions, labels, code)
- [ ] Spacing scale
- [ ] Border radii
- [ ] Shadows
- [ ] Icons
- [ ] CSS design tokens

## Core Components
- [ ] Buttons (primary, secondary, ghost, destructive, icon)
- [ ] Inputs (text, password, search, textarea)
- [ ] Select, Checkbox, Radio, Switch, Slider
- [ ] Avatar (single and group)
- [ ] Badges (category and status)
- [ ] Tooltip, Dropdown menu, Accordion
- [ ] Alert, Toast, Modal, Drawer
- [ ] Tabs, Breadcrumbs, Pagination

## Platform Components
{product-specific component checklist derived from the brief}

## Catalog Page
- [x] Shell: sidebar navigation, header, dark/light toggle
- [x] Foundations section
- [ ] Core components section
- [ ] Platform components section
```

For **Platform Components**: derive from the brief. An AI chat product gets: Prompt field, Model selector, Conversation messages, Generation indicator, Code block, Source card, Prompt chips, Generated image card, Upload area, Agent card, Task step, Approval request, Credit meter, Command palette, Empty states. A design tool gets different ones. A fintech gets different ones again. Never use a generic list.

### Phase 5 — Write design-system/PROMPT.md

The main brief document that an agent will read at the start of every work session. Structure:

```markdown
# Design System Brief — {project-name}

Build a reusable component library for {project-name}, {product description}.

Attach visual references before starting. Use them to define the color palette, typography, spacing, borders, icons, and component style. Turn the references into a single, consistent visual language.

In this phase, build the library and its catalog page. Product screens come later, assembled from these pieces.

---

## VISUAL FOUNDATIONS

Define and document:
- Background, surface, text, border, action, and feedback colors
- Typography for headings, body, captions, labels, and code
- Spacing scale, border radii, and shadows
- Consistent icon style

Organize these values as reusable CSS custom properties.

---

## CORE COMPONENTS

Build:
- Buttons: primary, secondary, ghost, destructive, icon button
- Inputs: text, password, search, textarea
- Select, Checkbox, Radio, Switch, Slider
- Avatar (single and group)
- Badges (category and status)
- Tooltip, Dropdown menu, Accordion
- Alert, Toast, Modal, Drawer
- Tabs, Breadcrumbs, Pagination

---

## PLATFORM COMPONENTS

Build:
{product-specific components, one per line, with a short description of behavior}

---

## BUILD RULES

Use the same foundations across all components.
Build larger components by composing smaller ones.

Include applicable states: default, hover, focus, selected, disabled, loading, error.
Ensure readable contrast and correct behavior with long text and on small screens.

Use realistic content in all examples — no lorem ipsum.

Build in Next.js (App Router) + TypeScript + Tailwind CSS + shadcn/ui. Start from shadcn/ui primitives and restyle them with the system's tokens. Map every CSS custom property into the Tailwind theme so components use token-based classes, never hardcoded values.

Make interactions work. Simulate AI responses and generation processes without relying on external APIs.

---

## MOTION & POLISH

Apply Emil Kowalski's principles to every component and screen.

### Easing tokens (required on :root)
```css
--ease-out: cubic-bezier(0.23, 1, 0.32, 1);
--ease-in-out: cubic-bezier(0.77, 0, 0.175, 1);
--ease-drawer: cubic-bezier(0.32, 0.72, 0, 1);
```

### Animation decision
Before any animation, ask: how often will users see this?
- 100+/day (keyboard shortcuts, command palette): no animation, ever
- Tens/day (hover, list navigation): remove or drastically reduce
- Occasional (modals, drawers, toasts): standard animation
- Rare / first-time (onboarding, celebrations): can add delight

### Duration
- Button press feedback: 100–160ms
- Tooltips, small popovers: 125–200ms
- Dropdowns, selects: 150–250ms
- Modals, drawers: 200–500ms
- Never exceed 300ms for everyday UI

### Easing
- **Entering** → `var(--ease-out)` — starts fast, feels responsive
- **On-screen movement** → `var(--ease-in-out)` — natural acceleration
- **Hover / color** → `ease`
- **Constant motion** → `linear`
- **Never `ease-in` in UI** — starts slow, makes interface feel sluggish

### Per-component rules
- **Button:** `transform: scale(0.97)` on `:active`, `transition: transform 160ms var(--ease-out)`
- **Popover / dropdown:** `transform-origin` from the trigger. Use `var(--transform-origin)` (Base UI). Modal stays `transform-origin: center`
- **Tooltip:** initial delay on first open; subsequent hovers skip delay and animation (`transition-duration: 0ms`)
- **Toast:** CSS transitions (not keyframes) for interruptibility. Exit should be faster than enter
- **Drawer:** use `var(--ease-drawer)` for the iOS-like curve
- **Lists:** stagger 30–80ms between items. Never block interaction during stagger

### Element entry
```css
/* Never */
.entering { transform: scale(0); }

/* Always */
.entering { transform: scale(0.95); opacity: 0; }

/* Modern CSS */
@starting-style { opacity: 0; transform: translateY(8px); }
```

### Performance
- Animate only `transform` and `opacity` — never `width`, `height`, `padding`, `margin`
- CSS transitions > keyframes for dynamic, interruptible UI
- For programmatic animations: WAAPI (`element.animate()`) — hardware-accelerated, no library
- `prefers-reduced-motion`: keep opacity/color, remove movement
- Hover states behind `@media (hover: hover) and (pointer: fine)` only

---

## CATALOG PAGE

The catalog lives at `app/page.tsx`, inside the shell in `components/catalog/`:
- Sticky sidebar with grouped navigation (drawer on mobile), highlighting the section in view
- Header with the project name, version and a dark/light theme toggle
- Groups in this order: Foundations → Core Components → one group per product area → Screens

Every piece gets an anchored section (`CatalogSection`) and an entry in `components/catalog/nav.ts`.
Show each component's name, variants, and states, followed by short usage notes.
The page should let you compare pieces, in both themes, and serve as a reference for building screens.
```

Adapt **Platform Components** to this product specifically.

### Phase 6 — Run /ds-tokens

Without asking, execute the `ds-tokens` skill immediately in the same session, using the brief and references already collected. Skip its own questions — everything it needs is in `CLAUDE.md` and `PROMPT.md`.

### Phase 7 — Build the catalog shell

Always build this — it is the fixed pattern of every design system made with this skill. Write all UI copy (nav labels, section titles, descriptions, aria-labels) in the language of the brief.

```bash
npx shadcn@latest add button sheet
```

**`components/catalog/nav.ts`** — the single source of the sidebar. Item `id`s are the anchors of the sections; items with `href` point to routes (screens).

```ts
export type CatalogNavItem = { id: string; label: string; href?: string };
export type CatalogNavGroup = { label: string; items: CatalogNavItem[] };

export const catalogNav: CatalogNavGroup[] = [
  {
    label: "Foundations",
    items: [
      { id: "brand", label: "Brand" },
      { id: "color", label: "Color" },
      { id: "typography", label: "Typography" },
      { id: "space-shape", label: "Space & shape" },
      { id: "icons", label: "Icons" },
      { id: "motion", label: "Motion" },
    ],
  },
  {
    label: "Core components",
    items: [
      { id: "buttons", label: "Buttons" },
      { id: "inputs", label: "Inputs" },
      { id: "selection", label: "Selection" },
      { id: "avatar-badges", label: "Avatar & badges" },
      { id: "menus-modals", label: "Menus & modals" },
      { id: "alerts-toasts", label: "Alerts & toasts" },
      { id: "navigation", label: "Navigation" },
      { id: "progress", label: "Progress" },
    ],
  },
  // One group per product area from the brief (e.g. "Landing page", "Patient area",
  // "Team dashboard", "AI assistant"), listing the platform components of STATUS.md.
  // { label: "Screens", items: [] } — /ds-screen adds { id, label, href } entries here.
];

export const catalogSectionIds = catalogNav.flatMap((group) =>
  group.items.filter((item) => !item.href).map((item) => item.id),
);
```

**`components/catalog/theme-toggle.tsx`** — swaps icons with the `dark:` variant, so it needs no `mounted` state and never mismatches on hydration.

```tsx
"use client";

import { Moon, Sun } from "lucide-react";
import { useTheme } from "next-themes";
import { Button } from "@/components/ui/button";

export function ThemeToggle() {
  const { resolvedTheme, setTheme } = useTheme();

  return (
    <Button
      variant="ghost"
      size="icon"
      aria-label="Toggle dark/light theme"
      onClick={() => setTheme(resolvedTheme === "dark" ? "light" : "dark")}
    >
      <Sun className="size-4 dark:hidden" aria-hidden />
      <Moon className="hidden size-4 dark:block" aria-hidden />
    </Button>
  );
}
```

**`components/catalog/catalog-shell.tsx`** — header + sticky sidebar on desktop, drawer on mobile, active-section highlight.

```tsx
"use client";

import { useEffect, useState, type ReactNode } from "react";
import { Menu } from "lucide-react";
import { Button } from "@/components/ui/button";
import { Sheet, SheetContent, SheetTitle, SheetTrigger } from "@/components/ui/sheet";
import { cn } from "@/lib/utils";
import { catalogNav, catalogSectionIds } from "./nav";
import { ThemeToggle } from "./theme-toggle";

function useActiveSection(): string {
  const [active, setActive] = useState(catalogSectionIds[0] ?? "");

  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        const visible = entries.find((entry) => entry.isIntersecting);
        if (visible) setActive(visible.target.id);
      },
      { rootMargin: "-20% 0px -70% 0px" },
    );
    for (const id of catalogSectionIds) {
      const element = document.getElementById(id);
      if (element) observer.observe(element);
    }
    return () => observer.disconnect();
  }, []);

  return active;
}

function CatalogNav({ active, onNavigate }: { active: string; onNavigate?: () => void }) {
  return (
    <nav aria-label="Design system" className="flex flex-col gap-6">
      {catalogNav.map((group) => (
        <div key={group.label} className="flex flex-col gap-1">
          <p className="px-3 pb-1 text-xs font-medium uppercase tracking-wide text-text-secondary">
            {group.label}
          </p>
          {group.items.map((item) => (
            <a
              key={item.id}
              href={item.href ?? `#${item.id}`}
              onClick={onNavigate}
              aria-current={active === item.id ? "location" : undefined}
              className={cn(
                "rounded-md px-3 py-1.5 text-sm text-text-secondary transition-colors hover:bg-surface-hover hover:text-text-primary focus-visible:outline-2 focus-visible:outline-border-focus",
                active === item.id && "bg-surface-active text-text-primary",
              )}
            >
              {item.label}
            </a>
          ))}
        </div>
      ))}
    </nav>
  );
}

export function CatalogShell({
  name,
  version,
  children,
}: {
  name: string;
  version: string;
  children: ReactNode;
}) {
  const active = useActiveSection();
  const [open, setOpen] = useState(false);

  return (
    <div className="flex h-dvh flex-col bg-bg-base text-text-primary">
      <header className="flex h-14 shrink-0 items-center gap-3 border-b border-border-default px-4 lg:px-6">
        <Sheet open={open} onOpenChange={setOpen}>
          <SheetTrigger
            render={<Button variant="ghost" size="icon" className="lg:hidden" aria-label="Open navigation" />}
          >
            <Menu className="size-4" aria-hidden />
          </SheetTrigger>
          <SheetContent side="left" className="w-72 overflow-y-auto p-4">
            <SheetTitle className="sr-only">Navigation</SheetTitle>
            <CatalogNav active={active} onNavigate={() => setOpen(false)} />
          </SheetContent>
        </Sheet>
        <span className="truncate font-semibold">{name}</span>
        <span className="shrink-0 rounded-full border border-border-default px-2 py-0.5 text-xs text-text-secondary">
          Design System v{version}
        </span>
        <div className="ml-auto">
          <ThemeToggle />
        </div>
      </header>
      <div className="flex min-h-0 flex-1">
        <aside className="hidden w-64 shrink-0 overflow-y-auto border-r border-border-default p-4 lg:block">
          <CatalogNav active={active} />
        </aside>
        <main className="min-w-0 flex-1 overflow-y-auto scroll-smooth px-4 py-10 motion-reduce:scroll-auto lg:px-10">
          {children}
        </main>
      </div>
    </div>
  );
}
```

`shadcn init` installs Base UI primitives by default, which compose through the `render` prop. If the project's `sheet` is built on Radix instead (`@radix-ui/react-dialog`), use `<SheetTrigger asChild><Button …><Menu … /></Button></SheetTrigger>`.

**`components/catalog/catalog-section.tsx`** — server components for groups and anchored sections.

```tsx
import type { ReactNode } from "react";

export function CatalogGroup({ title, children }: { title: string; children: ReactNode }) {
  return (
    <div className="mx-auto max-w-5xl pb-16">
      <h2 className="pb-8 text-3xl font-semibold tracking-tight">{title}</h2>
      {children}
    </div>
  );
}

export function CatalogSection({
  id,
  title,
  description,
  children,
}: {
  id: string;
  title: string;
  description?: string;
  children: ReactNode;
}) {
  return (
    <section
      id={id}
      aria-labelledby={`${id}-title`}
      className="scroll-mt-6 border-b border-border-default py-12 first:pt-0 last:border-0"
    >
      <h3 id={`${id}-title`} className="text-xl font-semibold">{title}</h3>
      {description && <p className="mt-1 max-w-prose text-sm text-text-secondary">{description}</p>}
      <div className="mt-6">{children}</div>
    </section>
  );
}
```

**`app/page.tsx`** — `<CatalogShell name="{Project name}" version="0.1">` wrapping one `CatalogGroup` per nav group and one `CatalogSection` per nav item (same `id`). Fill the Foundations now, rendered from the real tokens of `globals.css` (never re-typed values):
- **Brand** — logo/wordmark at several sizes (or the name set in the display font if there is no logo yet)
- **Color** — every semantic token as a swatch with its class/variable name and intent; primitive scales below
- **Typography** — each text style with font, weight and size; the full type scale
- **Space & shape** — spacing scale as proportional blocks with values, radii, shadow levels with token names
- **Icons** — a grid of the icon set with labels, naming the library and default size
- **Motion** — easing and duration tokens, with live demos (button press, tooltip, modal, drawer) that respect `prefers-reduced-motion`

Core and platform sections start with a one-line empty state naming what will live there; `/ds-component` replaces it with the real component. Adjust the font-size classes to the system's type tokens if `/ds-tokens` defined custom ones.

### Phase 8 — Verify

Run `npm run build`. If it fails, fix the cause and build again. Only report a failure if you can't fix it.

Check the catalog contract before moving on: the header has the theme toggle, every `nav.ts` item has a matching `CatalogSection` id, and the sidebar collapses into the drawer below `lg`.

### Phase 9 — Final summary

This is the first time the user hears from you since Phase 1. Reply with:

```
{project-name} is ready.

  Next.js app            — App Router + TypeScript + Tailwind + shadcn/ui (npm run dev)
  CLAUDE.md              — project context for future sessions
  STATUS.md              — component checklist to track progress
  design-system/
    PROMPT.md            — the main brief to open every work session
    references/          — visual references (add more anytime)
  app/globals.css        — tokens: primitives, semantics, dark mode, shadcn bridge
  app/page.tsx           — catalog: sidebar navigation, dark/light toggle, foundations

Build: ✓ npm run build passes. Run `npm run dev` to see it.

Decisions I made (change any of them):
  - Visual direction: {one line — derived from references or inferred from the brief}
  - Brand color: {value and why}
  - Fonts: {font pair and why}
  - Platform components: {n} items — {short list}

──────────────────────────────────────────
Full workflow (skills in recommended order):

  FOUNDATION
  /ds-tokens           → token architecture before any component
                         (primitive → semantic layers, dark mode, Emil's motion tokens)

  PER COMPONENT (repeat for each one)
  prototype            → explore 3 different directions before committing to one
  /ds-component        → build the component using established tokens
  break-ui             → stress-test with extreme data: long names, empty lists,
                         huge numbers, missing fields
  review-animations    → strict animation audit (Before/After table)
  /ds-audit            → full audit: tokens, states, accessibility
  /ds-docs             → document: when to use, props, ARIA, keyboard, do/don't

  SCREENS
  /ds-screen           → assemble a full screen from the system's pieces
  choreograph          → coordinate elements that animate together on screen
  mobile-native        → fix classic mobile web issues

  PROGRESS
  /ds-status           → track and update the checklist across sessions
──────────────────────────────────────────
```

End with one line: **"Tell me what you want to change, or start building components with `/ds-component`."**
