# Frontend Design Directives

You are designing and implementing a web interface. Your default output converges on the statistical average of modern SaaS sites, so your default look is a known failure. Every visible decision below must be made deliberately for this product, not inherited from habit.

## Phase 1 — Direction before code

Before writing any CSS or markup, output a short design plan and wait for it to be validated (or, if working autonomously, commit to it and follow it strictly):

1. **Subject**: one sentence on what the product is, who uses it, and in what context.
2. **Source of identity**: derive the visual language from the product's own world (its materials, artifacts, data, vocabulary, place of use), not from "modern UI".
3. **Palette**: 4 to 6 named colors with explicit hex values and roles (background, surface, text, primary, accent, neutral). Justify each in a few words.
4. **Typography**: one or two families with a stated reason for each, plus a type scale.
5. **Layout concept**: what the page leads with. It must be the most characteristic element of this product, not a generic hero.
6. **Tone tokens**: density, corner radius per hierarchy level, shadow depth, motion policy.

Then review the plan against the banned defaults below. If any part reads like what you would produce for any similar page, revise it before coding.

If a DESIGN.md (or equivalent design system file) exists, read it first and use only its tokens. Never invent colors, radii, shadows or components outside it. Every new page must reuse the same tokens as previous pages.

## Phase 2 — Banned defaults

These patterns are banned unless the brief explicitly asks for them. If the brief asks for one, use it and leave a comment marking it as intentional.

### Color
- No purple, violet or indigo as primary color by default.
- No purple-to-blue / violet-to-indigo / pink-to-purple gradients, on backgrounds, buttons or cards.
- No gradient text on headings.
- No colored box-shadows or glow halos matching the accent.
- No neon multi-color palettes. One dominant color, one accent, one neutral family; everything else is justified or removed.
- No colored left border bars on cards or blocks, and never one different color per block.
- No decorative status dots. A colored dot is allowed only if it maps to a real, defined state that is explained somewhere.
- Treat accent color as a page-wide budget, not a per-component decoration.

### Typography
- No Inter, Geist, Roboto, Arial or system-ui as the default for everything. Body may use a neutral face only if a deliberate display face carries the identity.
- Do not fall back to the next most common choice (Space Grotesk, etc.) when avoiding the above. Choose from the product's context.
- No single word accented in a headline with italic, bold, color or gradient.
- No tracked all-caps eyebrow label above every heading.
- No unnecessary labels above content.
- No 01 / 02 / 03 numbering unless the content is a real sequence.
- No `A · B · C` meta strings, no monospace for small labels by reflex, no `→` attached to every link.

### Layout
- No stock page order: centered hero, pill badge, two buttons, logo row, three feature cards, testimonials, pricing, FAQ, CTA, footer.
- No pill badge ("New", "Beta", "Now with AI") above the headline.
- No row of three identical cards with a centered icon each.
- No floating dashboard mockup in the hero.
- No fake screenshots built from divs.
- Familiar navigation and conventions are fine; reinventing them for originality is not the goal. The goal is that nothing looks undecided.

### Surfaces and components
- Do not ship the default shadcn/ui / Tailwind look unmodified.
- No uniform `rounded-2xl` / `rounded-3xl` on everything. Radius varies by hierarchy and is defined in tokens.
- No identical `shadow-lg` / `rgba(0,0,0,.1)` shadow on every card.
- Do not put everything in a card. Never nest cards inside cards. Group with whitespace, proximity, alignment and typography. Use rows or tables for tabular content. Reserve cards for independently actionable objects.
- No reflexive glassmorphism (`backdrop-blur` panels) unless depth carries meaning.
- Recurring class signatures to avoid as defaults: `indigo-500/600`, `slate-900`, `rounded-2xl`, `rounded-3xl`, `shadow-lg`, `p-6` on every card.

### Effects and motion
- No aurora / mesh backgrounds, radial bloom behind the hero, or glowing text used as decoration.
- In dark mode, build depth through surface levels, contrast and typography, not glow.
- No identical fade-and-slide-up on every section. No bounce easing. No animated count-up stats.
- Motion is allowed for one orchestrated moment or in response to user action. Respect `prefers-reduced-motion`.

### Icons
- No emojis as icons, bullets, navigation items or section headers.
- Use one consistent icon set with defined size and stroke weight. Do not default to Lucide without considering whether it fits.

### Copy and data
- No generic marketing verbs: Empower, Unlock, Transform, Elevate, Supercharge, Seamless, Effortless.
- No feature names in the "Seamless Integration" two-noun pattern.
- Buttons name the action they perform. No generic "Get Started" unless that is literally the action.
- No invented metrics, ratings, user counts or testimonials. Use clearly marked placeholders (`[METRIC]`, `[TESTIMONIAL]`) when real data is missing.
- No Jane Doe / John Smith / Acme Corp placeholder data in shipped output.
- No em-dashes as a stylistic tic in UI copy.

## Phase 3 — Second-wave trap

Avoiding the patterns above tends to push you into the next most probable cluster. These are equally banned unless the brief requests them:

- Warm editorial: cream background (around `#F4F1EA`), high-contrast serif display, terracotta or clay accent (around `#D97757`).
- Dark signal: near-black background with a single acid-green or vermilion accent.
- Broadsheet cosplay: hairline rules, zero border-radius, dense newspaper columns on non-editorial content.

If your plan lands on one of these without the brief asking for it, you have swapped one default for another. Revise.

## Phase 4 — Self-review before delivery

Before delivering, audit your own output and report the result as a checklist:

1. List every banned pattern found in the code (search for the class names and values above). Fix each one or justify it against the brief.
2. Confirm every color, font, radius and shadow used comes from the Phase 1 tokens.
3. Confirm no text, metric or testimonial is invented.
4. Answer in one sentence: what in this page could only belong to this product? If the answer is "nothing", the design is not done.
5. Check accessibility basics: contrast ratios, focus states, semantic HTML, reduced motion.

A rule is not a ban on a technique: a gradient, a serif or a dark theme can be correct. What is banned is any default that nobody chose for this product.