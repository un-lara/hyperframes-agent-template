# creative-qa

Review the scene plan against the established creative direction to confirm the plan executes the brief correctly before any HTML is written. This is a **quality gate**, not a generative step — the creative strategy was established in `creative-direction`. This skill checks whether the proposed plan honours it.

## Inputs required

- `CREATIVE-DIRECTION.md` — the authoritative creative brief
- `SCENE-PLAN.md`
- `resources/brief-summary.md`
- `design.md`
- `resources/source-flow-map.md` (if `source-extraction` ran)

## Steps

1. **Read all four input files in full.** Do not skim. The job of this skill is to spot mismatches between the brief and the plan — that requires holding both in working memory together.

2. **Review the scene plan on these dimensions.** For each, write a short pass/fail/concern note:

   a. **Narrative structure.** Does the scene order follow the creative brief's intent? Does it have a clear opening, body, and close appropriate for the video type from `CREATIVE-DIRECTION.md`?

   b. **Duration alignment.** Are individual scene durations consistent with the pacing prescribed for this video type? Examples:
   - Fast-cut marketing scenes should be ≤3 s each
   - Walkthrough scenes should give 5–10 s per content step (apply the 1.5× rule)
   - Motion graphics: minimum 1.5 s per data point
   - Talking head: defer to script timing

   c. **Key message coverage.** Does every key message from `brief-summary.md` appear in at least one scene? Are any critical beats missing? List explicitly which key message → which scene.

   d. **Creative conventions applied.** Does the scene plan make room for the type-specific creative techniques from `CREATIVE-DIRECTION.md`? Examples:
   - Software walkthroughs: zoom-focus scenes for individual elements
   - Marketing: kinetic text moments, opening attention-grab in the first 3 s
   - Motion graphics: sequenced data reveals
   - Talking head: lower-third and quote-callout moments

   e. **Vector recreation feasibility.** For walkthrough/UX-demo types where source screenshots will be recreated as vector HTML/CSS/SVG: is every screenshot in `source-flow-map.md` represented by a scene where it could plausibly be reconstructed? Flag any screenshot that's too dense or content-heavy to reconstruct meaningfully in the allocated duration — those scenes either need more time, need simplification, or need to be cut. (Reminder: screenshots are reference, not assets. `scene-author` will rebuild them as live HTML.)

   f. **Transitions planned.** Is there a logical flow between scenes, or are there jarring context shifts? Flag any seam where two adjacent scenes lack a visual or narrative bridge.

   g. **Redundancy check.** Are any scenes repetitive in content or visual approach? Two scenes covering the same key message with the same UI is often a sign one should be cut.

3. **Write `creative-qa-report.md`** at the project root. Structure:

   ```markdown
   # Creative QA Report

   **Scene plan version reviewed:** [date / hash if available]
   **Verdict:** Approved | Approved with suggestions | Revisions required

   ## What's working
   - [Specific positives — not generic praise]

   ## Gaps and misalignments
   - **Critical:** [Issues that block production — e.g. "Key message 'AML compliance' has no scene"]
   - **Advisory:** [Issues that would improve the video but aren't blockers — e.g. "Scene 04 at 4s feels tight for the data reveal"]

   ## Specific suggestions
   1. [Actionable change with the scene ID and a one-line rationale]
   2. ...

   ## Vector recreation flags
   - [Any screenshots that may be too dense to reconstruct in the allocated duration]
   - [Any scenes where the recreation strategy needs human input before scene-author can proceed]

   ## Verdict reasoning
   [One paragraph explaining the verdict.]
   ```

4. **Present the report to the human.** Ask: "Here's the creative QA. Want me to revise the scene plan based on these suggestions, or are you happy to proceed as is?"

5. **Loop or continue.**
   - If the human accepts the plan as-is: mark phase 6 complete in `PROJECT-STATE.json` and move on.
   - If the human wants revisions: loop back to `scene-planner` with the specific changes. Re-run this skill after the plan is updated.

## Outputs

- `creative-qa-report.md` — review against creative direction with prioritised suggestions

## Tools used

`Read`, `Write`.

## Success criteria

- Human has reviewed the creative QA report.
- Human has approved the scene plan for production (either accepting as-is or after a `scene-planner` revision loop).
- All critical gaps are either resolved or explicitly accepted by the human as known trade-offs.
- Any vector recreation flags are acknowledged before `scene-author` begins.
