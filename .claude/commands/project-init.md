# project-init

Run the conversational project kickoff, scaffold the HyperFrames project, merge CLAUDE.md files, and initialise all project state tracking before any production skill fires. This is always the first skill invoked — no other skill begins until this one completes.

## Inputs required

- None at invocation. Everything is collected through conversation in Part A.

## Steps

### Part A — Conversational intake

Open a conversation. Ask questions one at a time, in natural language, and wait for each answer before continuing. Do not present this as a form.

1. **"What type of video are you creating?"** — accept freeform; classify into one of: `product-walkthrough`, `simulation-walkthrough`, `ux-demo`, `feature-teaser`, `marketing-brand`, `feature-marketing`, `launch`, `text-motion-graphics`, `infographic-data`, `explainer`, `talking-head`, `cinematic-video`, `social-clip`. Hybrids are fine — record both.
2. **"What is your source material?"** — URL, simulation link, document, brief, or freeform description. If a URL/simulation, source-extraction will run later.
3. **"Is this a Traverse-branded video?"** — Yes → Traverse design system will be loaded by `design-system`. No → "Please share brand assets at any point before the design-system step, or I will extract them from the source."
4. **"Do you have a target duration in mind?"** — Optional. If not given, the agent will propose one in `creative-direction` based on the type.
5. *(Walkthrough types only — `product-walkthrough`, `simulation-walkthrough`, `ux-demo`)* **"Are you looking for an accurate, faithful rebuild of the source interaction — or a stylised representation that captures the narrative and feel without pixel-perfect fidelity?"** Default: `stylised`. Accurate mode is slower — it requires full traversal of the simulation. This sets the `accurate_or_stylised` flag.
6. **"Anything else I should know before I start? Characters, audience, tone, key messages?"** — Open-ended catch-all.

**Gate:** Do not proceed past Part A until questions 1, 2, and 3 are answered. For walkthrough types, question 5 is also required.

After the answers are in:

7. Classify the video type and determine which skills will fire for this run. Skills are conditional:
   - `source-extraction` — only if source is a URL/simulation/web product
   - `character-asset-generation` — only if characters are mentioned or expected
   - `voiceover-script` + `audio-production` — only if VO is needed (skip for silent/music-only)
   - All others always fire.

8. Confirm the plan back to the human in one short message: video type classification, skills that will fire, approximate phase order, any conditional skills flagged. Human confirms before scaffolding begins. If the human pushes back on the classification or skill plan, adjust before continuing.

### Part B — Project scaffolding

9. Run `npx hyperframes init` from the project root. This generates the HyperFrames project structure (`index.html`, `compositions/`, `meta.json`, `package.json`, `hyperframes.json`, and a HyperFrames-authored `CLAUDE.md`).

10. **CLAUDE.md merge — do NOT overwrite.** `npx hyperframes init` produces its own `CLAUDE.md` containing critical framework rules:
    - Required data attributes (`data-start`, `data-duration`, `data-track-index`)
    - `class="clip"` requirement on every timed element
    - `window.__timelines` registration pattern (NOT `window.__sceneTl` — that name is incorrect and will silently break rendering)
    - Deterministic-only logic rule (no `Date.now()`, no `Math.random()`, no network fetches)
    - `npm run check` mandate after every composition change
    - HyperFrames skill command table (`/hyperframes`, `/gsap`, `/hyperframes-cli`, etc.)
    - Documentation pointers (`npx hyperframes docs <topic>`, `https://hyperframes.heygen.com/llms.txt`)

    Steps to merge:
    a. Read the HyperFrames-generated `CLAUDE.md` (in the project root, just written by `npx hyperframes init`).
    b. Read the template `CLAUDE.md` that ships with this agent repo (sibling to `.claude/`).
    c. Produce a merged file: HyperFrames content preserved in full, then the agent's project context appended below it under a clear `## Motion Design Agent Context` heading.
    d. Write the merged result back to `CLAUDE.md`, overwriting the partial HyperFrames-only version.
    e. Verify the merged file contains: `window.__timelines`, `data-start`, `class="clip"`, `npm run check`, and the agent's persona section.

11. Run `npx hyperframes --version`. Record the version. If the version is `≤0.5.7`, flag to the human:

    > "HyperFrames v[X] has known issues with `data-composition-src` sequential switching. I'll use the inline `build-root.mjs` pattern instead — this is the only correct assembly approach in this version. Confirming I'll proceed."

    The inline build is always the default regardless of version — this flag exists so the human knows why.

12. Confirm the inline composition flag is locked. The agent uses `build-root.mjs` to produce a single `index.html`. Sub-composition switching via `data-composition-src` is not used.

### Part C — Project state initialisation

13. Write `PROJECT-STATE.json` at the project root, populated with the answers from Part A:

```json
{
  "project_name": "[name]",
  "video_type": "[classified type]",
  "source_material": "[URL or description]",
  "branding": "traverse" | "custom" | "pending",
  "target_duration_seconds": null | [number],
  "accurate_or_stylised": "accurate" | "stylised" | "n/a",
  "skills_plan": ["source-extraction", "resource-intake", "creative-direction", "..."],
  "current_phase": 0,
  "phase_gates": {}
}
```

The `accurate_or_stylised` field is `"n/a"` for non-walkthrough types.

14. Create the `resources/` directory structure at the project root:
    - `resources/screenshots/` (empty)
    - `resources/characters/` (empty)
    - `resources/brief-summary.md` (empty, to be populated by `resource-intake`)
    - `resources/acronym-list.md` (empty)
    - `resources/asset-manifest.json` (write `[]`)

## Outputs

- HyperFrames project structure (from `npx hyperframes init`)
- Merged `CLAUDE.md` at the project root — both HyperFrames framework rules and the Motion Design Agent context present
- `PROJECT-STATE.json` initialised with kickoff answers
- `resources/` directory structure as above

## Tools used

`Bash` (`npx hyperframes init`, `npx hyperframes --version`), `Read`, `Write`.

## Success criteria

- Human has answered questions 1–3 (plus question 5 for walkthrough types) and confirmed the skills plan.
- `npx hyperframes init` ran successfully.
- `CLAUDE.md` is the merged version. A grep for `window.__timelines`, `data-start`, `class="clip"`, and `npm run check` returns hits.
- `PROJECT-STATE.json` exists with all kickoff answers recorded.
- `resources/` directory structure exists with the five entries above.
- `current_phase` is `0` (this skill complete) ready for the next skill to advance it to `1`.
