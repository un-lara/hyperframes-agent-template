# scene-planner

Define the scene architecture with human-readable scene names, locked durations, and explicit confirmed/optional status. After this skill, scene durations are locked — voiceover script will be written to these durations, not the other way around.

## Inputs required

- `resources/brief-summary.md`
- `resources/source-flow-map.md` (if `source-extraction` ran)
- `resources/asset-manifest.json`
- `CREATIVE-DIRECTION.md` (for duration target and pacing rules)
- `design.md` (for any structural conventions)
- Target video duration from `PROJECT-STATE.json` or `CREATIVE-DIRECTION.md`

## Steps

1. **Read the inputs and assemble a candidate scene list.**

   For each scene, define:
   - **Scene ID** — `seq[NN]` (zero-padded, stable, never changes once issued)
   - **Human-readable name** — specific and descriptive: "EDD Escalation Review", "Marcus Video Call", "Dashboard Intro". Never generic ("Scene 3"). Never ambiguous ("Compliance Slide" if there are two compliance slides).
   - **Content summary** — what this scene shows and what it communicates (1–2 sentences)
   - **Source asset(s)** — which screenshots, characters, or other assets from `asset-manifest.json` it references
   - **Target duration** — seconds, informed by content complexity and the total duration budget from `CREATIVE-DIRECTION.md`
   - **Status** — `confirmed` or `optional`, with a one-line note for optionals explaining why they might be cut

2. **Source the scene list intelligently.**

   - If `source-flow-map.md` exists, use its sections as the candidate scene list. One flow-map section → one scene candidate (usually). Combine adjacent low-content sections; split heavy sections that exceed reasonable per-scene duration.
   - If no flow map (source was a brief, not a URL), derive scenes from the narrative arc in `brief-summary.md`. Use the key messages as anchors.
   - Always include an opening scene (sets context) and a closing scene (resolves or calls-to-action), even if the brief is light on these.

3. **Calibrate durations to the type from `CREATIVE-DIRECTION.md`.**

   Pacing rules by type:
   - Software/simulation/walkthrough: 5–10 s per content scene. Apply the 1.5× rule — give 1.5 s of screen time per second of VO talking points to let viewers read.
   - Marketing: 1.5–3 s per beat. Fast-cut.
   - Motion graphics / infographic: variable by data complexity — minimum 1.5 s per data point, never longer than the information needs.
   - Talking head / cinematic: driven by the script — defer to VO timing.

4. **Calculate the duration budget.**

   Present both totals:
   - **Confirmed total** — sum of all `confirmed` scene durations
   - **Total including optional** — sum of all scenes regardless of status

   If the confirmed total is outside ±10% of the target from `CREATIVE-DIRECTION.md`, flag it. Either propose scenes to add/cut, or revise the target with the human.

5. **Present the scene plan to the human for review.**

   Format as a markdown table:

   ```markdown
   | Seq | Name | What it shows | Source asset(s) | Duration | Status |
   |-----|------|--------------|-----------------|----------|--------|
   | 01  | dashboard-intro | Full dashboard, all widgets visible — sets context | seq01-dashboard-intro.png | 4.0s | confirmed |
   | 02  | edd-escalation-review | EDD case expanded, decision logic visible | seq02a-, seq02b- | 8.0s | confirmed |
   | 03  | marcus-video-call | Marcus joins via video, briefs on case | resources/characters/marcus.png | 6.5s | optional — only if the brief calls for human-in-loop framing |
   | ... |
   | TOTAL confirmed | | | | 87.5s | |
   | TOTAL inc. optional | | | | 94.0s | |
   ```

   Ask: "Does this scene architecture work? Want to rename anything, change durations, promote optional scenes, or reorder?"

6. **Iterate with the human** until the plan is approved. Common adjustments:
   - Renaming scenes for clarity
   - Promoting `optional` → `confirmed` or vice versa
   - Reordering scenes for narrative flow
   - Adjusting individual durations (the human may know a scene needs more breathing room)

7. **Write the approved plan to `SCENE-PLAN.md`** at the project root:

   ```markdown
   # Scene Plan — [Project name]

   **Locked:** [date]
   **Total runtime (confirmed):** [N]s
   **Total runtime (including optional):** [N]s

   ## Scenes

   [The full table from step 5, finalised]

   ## Notes
   - [Any context the human added during review]
   - [Any optional scenes flagged with their decision criteria]

   ## Lock policy
   These durations are locked for voiceover script calibration. Changes after this point require:
   1. Updating `SCENE-PLAN.md` first
   2. Re-running `voiceover-script` so the script re-targets the new durations
   3. Updating any composition files that already reference the old timing
   ```

8. **Update `PROJECT-STATE.json`** — set `current_phase` to 5 and add the locked total runtime.

## Scene naming convention

- Names must be **unique** within the project.
- Names must be **specific** enough to be unambiguous. "AFC Deficiency List" not "Compliance Slide". If a project has two compliance slides, both must be specifically named ("AFC Deficiency List" and "AFC Resolution Summary").
- Names use **lowercase kebab-case** when used as file identifiers (`seq02-edd-escalation-review`) and Title Case in the human-readable display.
- If a scene's duration or position changes after lock: update `SCENE-PLAN.md` first, then update all downstream references to match. Never mutate downstream without updating the plan.

## Outputs

- `SCENE-PLAN.md` — approved scene list with IDs, names, durations, status, source assets
- Updated `PROJECT-STATE.json` with locked runtime

## Tools used

`Read`, `Write`.

## Success criteria

- `SCENE-PLAN.md` exists with explicit human approval.
- Every scene is either `confirmed` or `optional`. No ambiguous status.
- Durations sum to a sensible total runtime — confirmed total within ±10% of the `CREATIVE-DIRECTION.md` target (or the human has explicitly waived the target).
- All scene names are human-readable, unique, and unambiguous.
- The lock policy is explicit in the file so future changes follow the right order.
