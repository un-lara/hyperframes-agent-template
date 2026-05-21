# resource-intake

Validate and catalogue all source material. Parse the brief. Extract key terminology, acronyms, and character definitions so downstream skills have a clean, structured base to draw from.

## Inputs required

- `resources/screenshots/` (from `source-extraction`, or manually supplied by the human)
- Project brief (document, URL, or freeform description from kickoff)
- Brand assets (logo, lockup — if supplied at kickoff)
- Any other reference materials provided

## Steps

1. **Completeness checklist.** Present a short status of what the agent has vs. what is still needed. Format:

   ```
   ✅ Brief: [path/description]
   ✅ Screenshots: [count] files in resources/screenshots/
   ⚠️ Brand assets: not supplied (will extract from source if Traverse=No)
   ⚠️ Voiceover preferences: not specified (will use defaults in voiceover-script)
   ```

   This is informational — do not block on missing items unless they are critical (no brief AND no source = blocker).

2. **Read and parse the brief.** Whether it's a document, URL, or freeform text, extract:
   - **Key messages** — the 3–7 most important points the video must communicate
   - **Narrative arc** — opening → body → close structure, if implied
   - **All acronyms and technical terms** — every `KYC`, `AFC`, `EDD`, `SaaS`, etc., with full forms where given or inferred
   - **Named characters and their roles** — flag for `character-asset-generation` (e.g. "Marcus, compliance officer")
   - **Explicit timing, pacing, or style requirements** (e.g. "should feel like a film trailer", "no longer than 60 seconds")
   - **Target video length** — if specified, record in `PROJECT-STATE.json` → `target_duration_seconds`
   - **Audience** — who this video is for, and any inferred tone implications

3. **Write `resources/brief-summary.md`.** Structure:

   ```markdown
   # Brief Summary

   ## Audience
   [who this video is for]

   ## Key messages
   1. ...
   2. ...

   ## Narrative arc
   [opening → body → close, or whatever structure the brief implies]

   ## Style/tone notes
   [any explicit requirements from the brief]

   ## Characters detected
   - [name]: [role]
   - ...

   ## Source citations
   [where in the brief each point came from — page numbers, section headings, paragraph refs]
   ```

4. **Write `resources/acronym-list.md`.** Every acronym with its full form. This list is queued for phonetic hyphenation in `voiceover-script` (e.g. `AFC` → `A-F-C`, `KYC` → `K-Y-C`). Format:

   ```markdown
   | Acronym | Full form | Phonetic (filled by voiceover-script) |
   |---------|-----------|---------------------------------------|
   | AFC     | Anti-Financial Crime | A-F-C |
   | EDD     | Enhanced Due Diligence | E-D-D |
   ```

5. **Character flagging.** If named characters were detected:

   > "I've detected [N] characters in the brief: [names]. I'll need avatar images for these — `character-asset-generation` will handle this after creative QA. I can help generate them in v1 by writing the prompts; you'll generate the images in ChatGPT/Midjourney and drop them into `resources/characters/`."

   If no characters: explicitly record "No characters detected" in `brief-summary.md` so the agent knows to skip `character-asset-generation` later.

6. **Update `resources/asset-manifest.json`** to be a complete catalogue of all source files:

   ```json
   [
     { "type": "screenshot", "seq": "01", "filename": "seq01-dashboard-intro.png", "content_source": "brief" },
     { "type": "brief", "filename": "project-brief.pdf", "parsed_to": "resources/brief-summary.md" },
     { "type": "brand-asset", "filename": "logo.svg", "status": "supplied" },
     { "type": "character-prompt", "name": "Marcus", "status": "pending" }
   ]
   ```

7. Note any missing assets as `pending` in the manifest, not as blockers — unless they are critical (no brief and no source). Critical blockers are escalated to the human; non-critical pending assets are noted and continued.

## Outputs

- `resources/asset-manifest.json` — complete catalogue of all source files
- `resources/brief-summary.md` — parsed interpretation of the brief
- `resources/acronym-list.md` — all acronyms with full forms, ready for phonetic formatting later
- Character detection summary (in `brief-summary.md`)

## Tools used

`Read`, `Write`, Claude in Chrome (`mcp__Claude_in_Chrome__navigate`, `get_page_text`) if the brief is a URL.

## Success criteria

- Brief parsed; key messages, narrative arc, and tone extracted.
- All known assets registered in `asset-manifest.json`.
- Any missing critical assets flagged to the human (not silently absent).
- Character detection complete — either characters listed with roles, or "No characters detected" explicitly recorded.
- `acronym-list.md` exists with every acronym from the brief, ready for phonetic hyphenation in `voiceover-script`.
