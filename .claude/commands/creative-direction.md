# creative-direction

Establish the complete creative strategy for this video before any design, scene planning, or content creation begins. This is the Creative Director's brief — it classifies the video type, applies type-specific best practices, determines ideal duration, and defines the visual and motion conventions all downstream skills must follow. Without this skill, the agent produces technically correct but creatively generic output.

## Inputs required

- `video_type` answer from `PROJECT-STATE.json` (or freeform text to reclassify)
- `accurate_or_stylised` flag from `PROJECT-STATE.json` (walkthrough types only)
- `resources/brief-summary.md`
- `resources/source-flow-map.md` (if `source-extraction` ran)
- Target audience and intended platform/use case (from the brief)

## Steps

### Step 1 — Video type classification

Classify the video into one of the following types (or a documented hybrid). The classification drives every downstream creative decision.

| Type | Description |
|------|-------------|
| `product-walkthrough` | Showing features of a software product or platform |
| `simulation-walkthrough` | Stepping through a guided simulation or scenario |
| `ux-demo` | Demonstrating a specific user flow or interaction pattern |
| `feature-teaser` | Short, punchy intro to a single new feature |
| `marketing-brand` | Promotional video — emotional, conversion-focused |
| `feature-marketing` | Marketing video centred on a specific feature |
| `launch` | New product or version announcement |
| `text-motion-graphics` | Typography-led, kinetic text, no or minimal imagery |
| `infographic-data` | Animated charts, stats, comparisons, data storytelling |
| `explainer` | Concept explanation through narration and illustration/motion |
| `talking-head` | Person speaking to camera; motion design is the support layer |
| `cinematic-video` | Actual video footage with motion design overlay |
| `social-clip` | Short-form (≤60s), vertical or square, thumb-stopping |

If the project is a hybrid (e.g. "simulation walkthrough with a brand outro"), record the primary type plus the hybrid override and note where the override applies.

### Step 2 — Duration target

Based on video type and use case, set the target duration and present it to the human for confirmation:

| Type | Short | Standard | Long |
|------|-------|---------|------|
| `feature-teaser` / `social-clip` | 15 s | 30 s | 60 s |
| `marketing-brand` / `feature-marketing` / `launch` | 30 s | 60–90 s | 120 s |
| `product-walkthrough` / `ux-demo` | 60 s | 90–120 s | 180 s |
| `simulation-walkthrough` | 90 s | 120–180 s | 240 s |
| `text-motion-graphics` / `infographic-data` | 30 s | 60 s | 90 s |
| `explainer` | 60 s | 120–180 s | 240 s |
| `talking-head` / `cinematic-video` | Varies — follows the script |

If `target_duration_seconds` was set in `PROJECT-STATE.json`, use that as a starting point and confirm it's appropriate for the type. If not set, propose one from the table.

### Step 3 — Accurate vs. stylised fidelity (walkthrough types only)

For `product-walkthrough`, `simulation-walkthrough`, and `ux-demo`, read the `accurate_or_stylised` flag from `PROJECT-STATE.json` and record it as a named parameter in `CREATIVE-DIRECTION.md`. This flag changes the creative approach for `scene-author`:

| Parameter | Scene author behaviour | Source extraction depth |
|-----------|----------------------|------------------------|
| `accurate` | Recreates exact UI layout, copy, and interaction states as faithfully as HyperFrames allows. Every on-screen element that matters to a real user must be represented. | Full traversal — every state captured and every piece of content confirmed. |
| `stylised` *(default)* | Captures narrative, flow, and key visual moments. Design language is applied over the source material — not constrained by it. Specific copy is taken from the brief or flow map; exact layout fidelity is not required. | Understand-and-infer — enough interaction to understand structure and purpose. |

This parameter also affects:
- **Typography treatment.** Accurate → use the source's actual text verbatim. Stylised → copy may be simplified, rewritten for clarity, or replaced with brief-sourced messaging.
- **Zoom/focus behaviour.** Accurate → zoom must land on the actual element being discussed. Stylised → zoom targets the most visually striking or narratively relevant area.
- **Transition conventions.** Both modes follow the type-specific transition rules below, but accurate mode keeps transitions tighter (shorter crossfades) to maintain tempo with the interaction.

### Step 4 — Creative conventions for this video type

Apply the relevant convention block to `CREATIVE-DIRECTION.md`. Tailor it to this specific project — do not paste the convention block generically.

---

**Software/simulation types** (`product-walkthrough`, `simulation-walkthrough`, `ux-demo`, `feature-teaser`):

- **Zoom behaviour.** Start with the full UI context visible. Zoom into the action area (button, form field, key data point) when an interaction occurs. Return to full-screen when navigating between sections. Zoom should be smooth, not a jump-cut. Zoom scale: typically 1.5–2× for element focus.
- **Focus annotation.** Use a visual callout ring or highlight to draw the eye to the element before the VO mentions it. The visual should lead the narration by 0.5–1 s.
- **Transitions.** Crossfades between scenes (0.4–0.6 s). Hard cuts are not appropriate. Between zoom states within a scene: use eased scale transitions, not cuts.
- **Typography.** Functional and clean. Scene titles as a lower-third overlay (not full-screen). Step numbers if the walkthrough is sequential. Callout labels directly adjacent to UI elements. No decorative typography.
- **Pace.** Measured — give viewers time to read what's on screen before the VO moves on. 1.5× rule: if the VO says something takes 3 s to explain, budget 4.5 s of screen time.
- **Colour/overlay.** Subtle brand overlay. The UI content must always be legible — overlays should not compete with the screen content.
- **Audio.** Voiceover-led at full volume. Background music at 15–25% volume maximum. Soft, non-distracting music bed.

---

**Marketing types** (`marketing-brand`, `feature-marketing`, `launch`, `social-clip`):

- **Movement.** Nothing should be static for more than 1.5 s. Use subtle scale drift (1.00 → 1.04 over 3 s), parallax layer movement, or element entrances/exits to maintain visual energy.
- **Transitions.** Dynamic and intentional. Match-cut timing (cut on movement, not between movements). Wipe reveals or mask reveals for headline moments. Crossfades only for emotional/slower sections.
- **Typography.** Typography IS the message. Large, bold headlines with animated reveals (character by character, word by word, or line by line depending on energy). Tracking and weight changes on emphasis words. Expressive — not functional.
- **Imagery vs. graphics.** Balance both. Graphics (shapes, colour blocks) for structural moments. Product UI screenshots or lifestyle imagery for evidence moments. Do not show walls of UI — select the most striking single element.
- **Colour.** High contrast. Brand colours at full saturation for key moments. Use colour as a transition device (colour block wipe).
- **Pacing.** Fast-cut structure. Each "beat" should be 1–3 s at most. The opening 3 s must earn the viewer's attention.
- **Audio.** Music-led. VO optional (may be text-on-screen only for social/silent autoplay). Music should drive the edit — cuts should feel rhythmically motivated.

---

**Motion graphics types** (`text-motion-graphics`, `infographic-data`):

- **Typography motion.** Text is the hero. Entrances should feel intentional — reveal (clip wipe), rise, or scale-in. Exits should feel resolved — scale down, fade, or slide out. Never let text just appear/disappear. Give each word or phrase a clear arrival and departure.
- **Hierarchy through motion.** The most important element in a given moment should be the one that moves. Use stillness to signal resolution and movement to signal attention.
- **Data visualisation.** Charts and graphs animate in sequence — do not reveal all data simultaneously. Bar charts grow up. Line graphs draw from left to right. Numbers count up. Give each data point at least 1.5 s of screen time.
- **Colour as meaning.** In infographic videos, colour must be functionally coded. Establish what each colour represents early and use it consistently. Don't use colour decoratively if it will confuse meaning.
- **Transitions.** Type-driven. Word reveals, line wipes, section changes driven by typography momentum (the last word of one section becomes the bridge to the next).
- **Duration.** Short — respect the viewer. Each piece of information should have exactly the screen time it needs and no more.

---

**Talking head / cinematic types** (`talking-head`, `cinematic-video`):

- **Motion design role.** Support layer only — not the hero. Lower thirds, key quote callouts, progress indicators, B-roll transition graphics.
- **Lower thirds.** Clean, minimal. Brand colour bar or line. Name + role in two-weight typography. Enters on a slide or reveal. Stays for 3–4 s then exits cleanly.
- **Key quote callouts.** Pull the most quotable 5–7 words from the VO and animate them as a text overlay during that moment. Large, weighted, fades with the speech.
- **Transitions.** Match-cut if editing video footage. Graphic wipes for B-roll section breaks. Do not over-design — the footage is the content.

---

### Step 5 — Output: CREATIVE-DIRECTION.md

Write `CREATIVE-DIRECTION.md` at the project root. It should be compact and reference-ready. Structure:

```markdown
# Creative Direction — [Project name]

## Classification
- Primary type: [type]
- Hybrid notes: [if any]
- Rationale: [why this classification, in one paragraph]

## Duration target
- Target: [N] seconds ([Short | Standard | Long])
- Rationale: [why this length for this content]

## Fidelity mode (walkthrough types only)
- accurate_or_stylised: [accurate | stylised | n/a]
- What this means for this project: [one paragraph]

## Creative conventions
[The relevant convention block from Step 4, tailored to this project. Replace generic examples with project-specific calls.]

## Project-specific creative decisions
[E.g. "Apply marketing-brand conventions to the final 10 s outro even though this is a simulation walkthrough."]

## Creative brief (the 30-second read)
[A plain-language paragraph the human can read to understand the creative intent in 30 seconds. This is the elevator pitch for the video — write it last.]
```

### Step 6 — Human confirmation

Present `CREATIVE-DIRECTION.md` to the human. Ask: "Does this classification, duration, and creative approach feel right? Anything to adjust before I move to design?" This is a fast check, not a detailed review. If the human adjusts, revise the file and re-present.

## Outputs

- `CREATIVE-DIRECTION.md` — the creative brief that all downstream skills reference

## Tools used

`Read`, `Write`.

## Downstream impact

Every subsequent skill reads `CREATIVE-DIRECTION.md`:

- `design-system` calibrates motion rules and typography approach from it
- `scene-planner` uses it for duration allocation and scene energy pacing
- `creative-qa` uses it as the benchmark for its review
- `scene-author` uses it for animation style decisions (zoom, transition, typography) and the accurate/stylised fidelity level
- `voiceover-script` uses it for pacing and tone calibration

## Success criteria

- Human has confirmed the video type classification, duration target, creative approach, and (for walkthrough types) accurate/stylised mode.
- `CREATIVE-DIRECTION.md` exists and reads as a standalone brief.
- The `accurate_or_stylised` parameter is explicitly named in `CREATIVE-DIRECTION.md` for walkthrough types.
- The creative brief paragraph at the end can be read in under 30 seconds and conveys the project intent.
