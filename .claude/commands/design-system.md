# design-system

Define the complete visual and motion language for the project, with a visual preview so the human can approve by seeing rather than reading tokens.

## Inputs required

- `resources/brief-summary.md`
- Brand assets (logo, lockup, colour-specified materials — if supplied)
- `branding` field from `PROJECT-STATE.json` (`traverse` | `custom` | `pending`)
- `CREATIVE-DIRECTION.md` (for motion-rule calibration to the video type)

## Steps

### Step 1 — Branding logic

Apply this decision flow based on `PROJECT-STATE.json` → `branding`:

```
Is this a Traverse-branded video?
   YES (traverse) → Load Traverse standard design system
                    (standard palette tokens, Plus Jakarta Sans / Inter,
                     standard motion rules)
                    Human may override specific tokens if needed.
   NO  (custom)   → Extract brand system from supplied assets.
                    If no assets supplied: ASK the human to supply before
                    proceeding. Do not invent brand identity.
   PENDING        → Block and ask the human to confirm branding before
                    continuing. This skill cannot complete without a branding decision.
```

### Step 2 — Traverse standard design system (when branding=traverse)

If branding is Traverse, load the standard token set:

```css
/* Palette */
--surface-base: #FFFFFF;
--surface-elevated: #F7F8FA;
--surface-deep: #1A1D29;
--text-primary: #1A1D29;
--text-secondary: #5C6478;
--text-on-dark: #FFFFFF;

/* Brand */
--primary: #2D6BFF;       /* Traverse blue — actions, CTAs, key accents */
--secondary: #7C3AED;     /* Purple — supporting accent */
--accent: #FF6B47;        /* Coral — highlights, focus rings */

/* Semantic */
--danger: #DC2626;
--warning: #F59E0B;
--success: #10B981;

/* Typography */
--font-display: "Plus Jakarta Sans", -apple-system, sans-serif;
--font-body: "Inter", -apple-system, sans-serif;
```

(These are the agreed Traverse defaults. If the project requires overrides, capture them and apply on top — do not silently mutate the standard token set.)

### Step 3 — Custom brand extraction (when branding=custom)

If brand assets were supplied:

1. Read the logo/lockup file. Extract dominant colours (top 4–6 by usage).
2. Read any supplied style guide. Pull primary, secondary, accent, surface, text colours.
3. Identify the brand typeface(s) — if not declarable as web-safe, propose a near-match (e.g. Söhne → Inter).
4. Record everything in `design.md`.

If brand assets were NOT supplied and branding is `custom`:

> "I need brand assets before I can set up the design system. Please share logo, colour palette, or any existing style guide. If you don't have these, I can derive a palette from the source URL — say 'extract from source' and I'll proceed."

If the human says "extract from source": use Claude in Chrome to inspect the source's computed styles and pull dominant colours and typography.

### Step 4 — Motion rules (calibrated to creative direction)

Read `CREATIVE-DIRECTION.md` to determine motion calibration. Different video types get different defaults:

**Software/simulation types** — measured easing, longer transitions:
```
--ease-default: cubic-bezier(0.4, 0, 0.2, 1)    /* gentle ease-in-out */
--duration-fast: 0.3s
--duration-default: 0.5s
--duration-slow: 0.8s
--crossfade: 0.5s
--click-cue-lead: 0.6s   /* visual leads VO by 0.6s */
```

**Marketing types** — punchier easing, faster:
```
--ease-default: cubic-bezier(0.16, 1, 0.3, 1)   /* expo out */
--duration-fast: 0.2s
--duration-default: 0.35s
--duration-slow: 0.6s
--crossfade: 0.3s
--match-cut: 0.0s        /* zero-frame cuts on movement */
```

**Motion graphics / explainer** — typography-driven:
```
--ease-default: cubic-bezier(0.16, 1, 0.3, 1)
--duration-fast: 0.25s
--duration-default: 0.4s
--duration-slow: 0.7s
--text-reveal-stagger: 0.05s  /* per character */
--line-reveal-stagger: 0.15s  /* per line */
```

### Step 5 — Write design.md

Compile the full design system into `design.md` at the project root:

```markdown
# Design System — [Project name]

## Branding source
[Traverse standard | Custom brand from {supplied assets / source extraction}]

## Palette tokens
[Full token list with hex values, grouped: surface, brand, semantic, text]

## Typography
- Display: [stack and size scale]
- Body: [stack and size scale]
- Size scale: [e.g. xs 12 / sm 14 / base 16 / lg 20 / xl 28 / 2xl 40 / 3xl 64]
- Weight scale: [e.g. 400 / 500 / 600 / 700]

## Motion rules
[Easing, durations, crossfade, click-cue lead, calibrated to the video type from CREATIVE-DIRECTION.md]

## Scene-specific animation patterns
- Entrances: [the default entrance pattern — e.g. opacity + 16px upward translation, 0.5s expo-out]
- Exits: [default exit pattern]
- Emphasis: [how key elements get attention — scale, weight change, colour shift]
- Zoom: [if applicable — scale range, easing, duration]

## Locked
This design is locked once approved. Any revision requires explicit human re-approval. Do not silently mutate tokens during scene authoring.
```

### Step 6 — Generate visual preview

Generate 1–2 mockup frames showing the design system applied to a sample scene layout. These can be:

- An HTML file at `design-preview/preview-01.html` with a representative scene mockup (typographic hierarchy, colour use, sample motion via CSS animation or static frames)
- A second preview at `design-preview/preview-02.html` showing a different scene archetype (e.g. one screen-focused, one typography-focused)

The preview is what the human reviews — not the tokens list. The goal is for them to see "yes, that's the visual direction" or "no, push it warmer / cooler / bolder" without parsing hex values.

Use `Write` to create the HTML files. They should be standalone (no external CDNs unless tailwind is preconfigured for the project — check `hyperframes.json`).

### Step 7 — Human review and approval

Present:
1. `design.md` (skim-readable)
2. The visual preview file(s) — give the human the path to open them

Ask: "Here's the design — open the preview to see it applied. Approve, or tell me what to change."

If changes requested: update `design.md`, regenerate the preview, present again. Loop until approved.

### Step 8 — Write ANIMATION-RULES.md

Once `design.md` is approved, extract just the motion rules into a separate machine-readable file `ANIMATION-RULES.md`. `scene-author` reads this file for per-scene animation defaults — keeping it separate from the broader `design.md` makes it cheaper to load into scene-author's context.

```markdown
# Animation Rules

## Easing
--ease-default: [value]
--ease-emphasis: [value]
--ease-exit: [value]

## Durations
--duration-fast: [value]
--duration-default: [value]
--duration-slow: [value]
--crossfade: [value]
--click-cue-lead: [value]

## Patterns
- Element entrance: [exact pattern, in plain language: "opacity 0 → 1, y +16px → 0, duration-default, ease-default"]
- Element exit: [exact pattern]
- Emphasis: [exact pattern]
- Scene crossfade: [exact pattern — half before, half after the seam]

## Forbidden
- No non-deterministic timing (no Date.now, no random delays)
- No animations longer than 1.2s on a single element unless explicitly required
```

## Outputs

- `design.md` (approved, locked)
- `ANIMATION-RULES.md` (machine-readable, referenced by `scene-author`)
- `design-preview/preview-[N].html` (visual mockup frames)

## Tools used

`Read`, `Write`, Claude in Chrome (if extracting from source). For visual previews, write HTML directly to the file.

## Success criteria

- Human has reviewed the visual preview and approved the design.
- `design.md` and `ANIMATION-RULES.md` exist at the project root.
- Motion rules in `ANIMATION-RULES.md` are calibrated to the video type from `CREATIVE-DIRECTION.md` (not generic defaults).
- Design is marked locked — no revisions without explicit human re-approval.
- If branding was `custom`, the source of the brand tokens is recorded in `design.md` (supplied assets, source extraction, or both).
