# Frontend Remediation Directives

You are fixing an existing web interface that shows the generic look produced by AI tools. Your task is not to redesign from scratch, but to identify every default nobody chose, replace it with a deliberate decision for this product, and preserve everything that works.

## Hard constraints

- Preserve all functionality, routes, component APIs, props, events, state logic and data flow.
- Preserve content meaning. Rewrite copy only where it falls under the copy rules below.
- Preserve or improve accessibility: semantic HTML, ARIA, focus order, keyboard navigation, contrast.
- Do not change the framework, component library or build setup unless explicitly asked.
- Keep existing brand assets (logo, mandated brand colors, licensed fonts). A brand color that the product actually owns is a decision, not a default.
- Work in small, reviewable changes. Do not rewrite files wholesale when a targeted edit suffices.

## Phase 1 — Audit before touching anything

Scan the codebase (CSS, Tailwind classes, theme config, components, templates, copy) and produce an audit report. For each finding give: file and line, the pattern, and a priority.

Priorities:
- **P0** — visible to any visitor in under a second (gradient headline, stock hero, three icon cards, invented metrics, emojis as icons).
- **P1** — visible to a designer or developer (default shadcn look, uniform radius and shadow, nested cards, eyebrow labels, motion on every section).
- **P2** — craft and polish (inconsistent spacing scale, icon stroke mismatch, minor token drift).

Patterns to search for:

### Color
- Purple, violet or indigo as primary color without brand justification.
- Purple-to-blue / violet-to-indigo / pink-to-purple gradients on backgrounds, buttons, cards.
- Gradient text on headings (`bg-clip-text`, `text-transparent`, `-webkit-background-clip: text`).
- Colored box-shadows or glow halos matching the accent.
- Neon multi-color palettes with no hierarchy.
- Colored left border bars on cards, especially one different color per block.
- Status dots that map to no real state.
- Recurring classes: `indigo-500/600`, `violet-*`, `purple-*`, `from-* to-*` gradients, `slate-900` as default dark.

### Typography
- Inter, Geist, Roboto, Arial or system-ui used for everything with no display face.
- A single word accented in a headline (italic, bold, color, gradient).
- Tracked all-caps eyebrow labels above headings (`uppercase tracking-wide/widest`).
- Unnecessary labels above content.
- 01 / 02 / 03 numbering on non-sequential content.
- `A · B · C` meta strings, reflexive monospace for small labels, `→` attached to every link.

### Layout
- Stock page order: centered hero, pill badge, two buttons, logo row, three feature cards, testimonials, pricing, FAQ, CTA, footer.
- Pill badge ("New", "Beta", "Now with AI") above the headline.
- Row of three identical cards with a centered icon each.
- Floating dashboard mockup in the hero.
- Fake screenshots built from divs.

### Surfaces and components
- Unmodified shadcn/ui / Tailwind default appearance.
- Uniform `rounded-2xl` / `rounded-3xl` everywhere.
- Identical `shadow-lg` / `rgba(0,0,0,.1)` on every card.
- Everything wrapped in cards; cards nested in cards.
- `p-6` + `rounded-2xl` + `shadow-lg` repeated as a signature combo.
- Reflexive glassmorphism (`backdrop-blur`) with no depth meaning.

### Effects and motion
- Aurora / mesh backgrounds, radial bloom behind the hero, glowing text.
- Identical fade-and-slide-up on every section, bounce easing, animated count-up stats.
- Missing `prefers-reduced-motion` handling.

### Icons
- Emojis used as icons, bullets, navigation items or section headers.
- Mixed or unconsidered icon sets, inconsistent size and stroke.

### Copy and data
- Generic verbs: Empower, Unlock, Transform, Elevate, Supercharge, Seamless, Effortless.
- Two-noun feature names ("Seamless Integration", "Smart Analytics").
- Generic "Get Started" buttons that do not name the action.
- Invented metrics, ratings, user counts, testimonials.
- Jane Doe / John Smith / Acme Corp placeholder data.
- Em-dashes used as a stylistic tic in UI copy.

### Second-wave defaults
Flag these too, unless the brief or brand requires them:
- Cream background (around `#F4F1EA`) + high-contrast serif + terracotta/clay accent (around `#D97757`).
- Near-black background + single acid-green or vermilion accent.
- Hairline rules, zero radius, dense newspaper columns on non-editorial content.

Do not modify anything during this phase. Deliver the audit first.

## Phase 2 — Direction

Before fixing, write a short direction (or read the existing DESIGN.md if one exists and flag where the code diverges from it):

1. **Subject**: what the product is, who uses it, in what context.
2. **What to keep**: elements of the current interface that are deliberate, brand-owned or working well. List them explicitly.
3. **Source of identity**: derive the visual language from the product's own world (its data, materials, artifacts, vocabulary, place of use).
4. **Palette**: 4 to 6 named colors with hex values and roles. Reuse brand colors where they exist.
5. **Typography**: one or two families with a reason for each, plus a type scale.
6. **Lead element**: what the page should open with — the most characteristic thing about this product.
7. **Tone tokens**: density, radius per hierarchy level, shadow depth, motion policy.

Check the direction against the second-wave defaults. If it lands on one without justification, revise.

Write the result to DESIGN.md (create it if absent) and centralize the tokens in the theme config or CSS custom properties. All fixes must reference these tokens; no hard-coded values.

## Phase 3 — Fix in priority order

1. Fix all P0 findings first, then P1, then P2.
2. Replace hard-coded colors, radii, shadows and fonts with tokens.
3. Remove gradients, glows, glass and decorative motion that carry no meaning. Keep at most one deliberate motion moment.
4. Flatten nested cards. Regroup content with spacing, alignment and typography; use rows or tables for tabular data.
5. Replace emojis with one consistent icon set at defined size and stroke.
6. Rewrite generic copy: buttons name their action, features describe what they actually do.
7. Replace invented metrics and testimonials with clearly marked placeholders (`[METRIC]`, `[TESTIMONIAL]`) and list them in the report. Never invent replacements.
8. If a flagged pattern is intentional per brand or brief, keep it and add a comment marking it as deliberate.

Do not introduce new defaults while removing old ones. Replacing purple with terracotta, or Inter with Space Grotesk, is not a fix.

## Phase 4 — Verify and report

1. Re-run the Phase 1 scan. Report remaining findings; there must be no unjustified P0 or P1.
2. Confirm no functionality, route, prop or accessibility feature regressed.
3. Confirm every color, font, radius and shadow in use comes from the tokens.
4. List all placeholders inserted for missing real data.
5. Answer in one sentence: what in this interface could now only belong to this product? If the answer is "nothing", the remediation is not done.

Deliver: the audit report, the direction (DESIGN.md), the diff grouped by priority, and the final verification checklist.

A pattern is not banned as a technique: a gradient, a serif or a dark theme can be correct. What gets removed is any default that nobody chose for this product.