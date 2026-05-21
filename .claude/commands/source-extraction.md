# source-extraction

Navigate the source with enough depth to establish clear understanding of every section, capture screenshots of all key states, cross-reference content against any supplied briefing material, and map the full narrative flow — so scene-author has everything needed to animate each scene without returning to the source.

**Conditional:** Only invoked when source material is a URL, browser-based simulation, or live web product.

**Do not confuse this skill with `/website-to-hyperframes`.** That is a HyperFrames-native skill for the one-URL-in, video-out marketing pipeline. `source-extraction` is the Motion Design Agent's methodical documentation skill for capturing simulation states individually for animation. They serve different purposes — never substitute one for the other.

## Inputs required

- Source URL or simulation link (from `PROJECT-STATE.json` → `source_material`)
- `accurate_or_stylised` mode flag (from `PROJECT-STATE.json`)
- Any briefing documents, scripts, or reference materials supplied at kickoff
- Any guidance on the primary user journey or narrative path

## Steps

### Part A — Navigation discovery and source understanding

1. **Open the source using Claude in Chrome.** Use `mcp__Claude_in_Chrome__navigate` to load the URL. `web_fetch` is insufficient for JS-rendered sources (Genially, SPAs, simulations, interactive tools) — it returns a loading shell. Live browser rendering with JavaScript execution is required.

2. Discover navigation generically by inspecting what is clickable, scrollable, or interactive on the page. Use `get_page_text`, `read_page`, and `find` to map the structure. Do NOT assume any specific app structure, slide framework, or navigation pattern — every source is treated as unknown until explored.

3. **Understand-and-infer approach.** Interact with the source to the depth required to establish clear understanding of:
   - **Structure** — how many sections/screens/states exist and how they are organised
   - **Content** — what each section communicates, what data or copy it contains
   - **Purpose** — what role each section plays in the overall narrative

   Exercise judgment about when you have understood enough. The test: *could scene-author animate this section without returning to the source?* If yes, move on. If no, interact further.

4. Do not move on from any section until you understand its content, purpose, and transitions. If a section requires scrolling, additional clicks, or expanded states to surface its full content, do that before moving on.

### Part B — Screenshot capture and sequencing

5. For each distinct section, screen, or narrative state:
   - Take a screenshot using `mcp__Claude_in_Chrome__computer` (screenshot action).
   - Name it: `seq[NN]-[descriptive-name].png` (zero-padded sequence number, kebab-case name).
   - Record: the trigger/action that entered this state, a description of what is visible, the exit condition or next state.

6. If multiple screenshots are needed for one section (e.g. before/after an interaction, or a scroll state), name them: `seq[NN]a-[name].png`, `seq[NN]b-[name].png`, etc.

7. Handle branching paths: identify the primary narrative path and explicitly flag alternatives in the flow map. Do not silently choose one branch over another without noting it.

### Part C — Content cross-referencing

8. For each captured section, determine what the accurate, factual content should be for scene authoring. Try each priority in sequence:

   - **Priority 1 — Briefing document.** If a brief, script, deck, or reference document was supplied at kickoff, cross-reference this section against the brief. Extract authoritative copy, key messages, data points, and terminology from the brief. This is the highest-confidence source — use it when available.

   - **Priority 2 — Screenshot / screen extraction.** If no brief was supplied, or the brief does not cover this section, interact with the simulation or page sufficiently to read the actual on-screen text and data. Zoom in or scroll if text is not legible at screenshot level. Use `get_page_text` to extract text content. Record the visible content.

   - **Priority 3 — Flag and escalate.** If neither a brief nor legible screen content is available (JS-rendered text that is inaccessible, a section the brief doesn't mention, or content hidden behind an interaction you cannot complete), flag it explicitly:

     ```
     ⚠️ CONTENT NOT CONFIRMED — [section name]
     Source: neither brief nor screenshot provides clear content for this scene.
     Scene author will require human input or placeholder content before authoring.
     ```

   Record the flag in the flow map. Do not silently proceed with guessed content.

9. Each section in the flow map records its content source: `brief` | `screenshot` | `⚠️ flagged`.

### Part D — Accurate vs. stylised mode

10. Apply the `accurate_or_stylised` mode flag from `PROJECT-STATE.json`:

    - **Accurate mode.** Capture exact UI states, precise copy, specific values, full interaction sequences, and the exact behaviour of every element. Scene-author will recreate the source faithfully. Requires full traversal of all relevant states. No inferring — if you cannot confirm a value, flag it.

    - **Stylised mode (default).** Focus on the narrative structure, key visual moments, and what each section communicates. Interaction details and exact copy are inferred and interpreted; capture enough to animate the *idea* of each section, not a pixel-perfect recreation. Unconfirmed details are acceptable in stylised mode — note them but do not flag them as critical.

### Part E — Flow map, confirmation, and registration

11. Produce `resources/source-flow-map.md` — an ordered table of all sections, with for each row:
    - Sequence number and human-readable name
    - Description of what is shown and what it communicates
    - Trigger/action that enters this state
    - Content source: `brief` | `screenshot` | `⚠️ flagged`
    - Mode: `accurate` | `stylised`
    - Any content flags from Part C

    Markdown table format:

    ```markdown
    | Seq | Name | What it shows | Trigger | Content source | Mode | Flag |
    |-----|------|--------------|---------|---------------|------|------|
    | 01  | dashboard-intro | Full dashboard, all widgets visible | Page load | brief | stylised | — |
    | 02a | edd-escalation-review-collapsed | EDD case list, collapsed | Click "EDD" tab | screenshot | stylised | — |
    | 02b | edd-escalation-review-expanded | First case expanded, full detail | Click first row | screenshot | stylised | ⚠️ amount field illegible |
    ```

12. Present the flow map to the human. Ask: "Does this capture all the key sections? Are there any interactions I missed? Here are [N] content flags that will need human input before scene authoring."

13. After human confirmation:
    - Copy all screenshots into `resources/screenshots/`.
    - Register every screenshot in `resources/asset-manifest.json` with its sequence number, filename, content source, and any flags.
    - List all content flags in a summary block for human awareness before Phase 2 begins.

## Outputs

- `resources/screenshots/seq[NN]-[name].png` — sequentially named, ordered
- `resources/source-flow-map.md` — all sections with descriptions, triggers, content source, mode, flags
- Updated `resources/asset-manifest.json` — every screenshot registered

## Tools used

Claude in Chrome (`mcp__Claude_in_Chrome__navigate`, `mcp__Claude_in_Chrome__get_page_text`, `mcp__Claude_in_Chrome__read_page`, `mcp__Claude_in_Chrome__find`, `mcp__Claude_in_Chrome__computer` for screenshots), `Write`, `Read`.

**Do NOT use `mcp__workspace__web_fetch` for the source.** It returns a loading shell for JS-rendered pages and will silently produce empty content. If a fetch returns only navigation chrome with no body content, the source is client-rendered — use Claude in Chrome instead.

## Success criteria

- Human has confirmed the flow map is complete and the screenshots cover all necessary states.
- All screenshots are present and correctly named in `resources/screenshots/`.
- `asset-manifest.json` reflects all screenshots with correct sequence numbers.
- All content flags from Part C are acknowledged by the human (not necessarily resolved — some may be deferred to scene authoring).
- The `accurate_or_stylised` mode is recorded against every entry in the flow map.
