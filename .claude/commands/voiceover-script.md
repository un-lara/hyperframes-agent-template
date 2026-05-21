# voiceover-script

Write a production-ready, timing-calibrated TTS script with phonetic formatting built in. The script is written to the video timing — the video is not re-timed to the script.

**Conditional:** Only invoked if `voiceover` is in the project's skill plan. Skip for silent / music-only videos.

## Inputs required

- `SCENE-PLAN.md` (scene durations are the AUTHORITATIVE timing targets — never change them to fit the script)
- `resources/acronym-list.md` (from `resource-intake`)
- `resources/brief-summary.md` (for key messages and tone)
- `CREATIVE-DIRECTION.md` (for pacing and tone calibration per video type)

## Steps

### Step 1 — Read the inputs

Hold the scene plan, key messages, and creative direction in working memory together. The job is to write narration that:
- Hits the locked scene durations within ±10%
- Communicates the key messages in the order the scenes plan
- Reads naturally at ~150 wpm (baseline TTS rate for ElevenLabs default voices)
- Uses phonetic hyphenation for every acronym so the TTS pronounces it letter-by-letter

### Step 2 — Apply phonetic hyphenation to acronyms

For every entry in `resources/acronym-list.md`, fill in the phonetic column:

| Acronym | Spoken letter-by-letter |
|---------|------------------------|
| AFC | A-F-C |
| KYC | K-Y-C |
| EDD | E-D-D |
| SaaS | sass *(pronounced as a word — flag this kind as "pronounced word" not letter-by-letter)* |

For acronyms that ARE pronounced as words (NASA, SaaS, GIF), do NOT hyphenate — they read naturally. Flag in `acronym-list.md` which approach each acronym uses.

Update `resources/acronym-list.md` with the filled-in phonetic column.

### Step 3 — Calibrate the baseline rate

Baseline rate: **~150 wpm** (2.5 words per second) for ElevenLabs at default stability/style.

Per-scene word budget = `scene_duration_seconds × 2.5`, with ±10% tolerance.

| Scene duration | Word budget | Tolerance |
|---------------|-------------|-----------|
| 4 s | 10 words | 9–11 |
| 6 s | 15 words | 13–17 |
| 8 s | 20 words | 18–22 |
| 12 s | 30 words | 27–33 |

For accurate-mode walkthroughs, dial slightly slower (140 wpm) — viewers need extra time to absorb UI detail.
For marketing/social, dial slightly faster (165 wpm) — energy demands it.
For motion graphics: write the minimum words needed to support the visual; let stillness do work.

### Step 4 — Write narration per scene

For each confirmed scene in `SCENE-PLAN.md`:

1. Read the scene's content summary, key message coverage, and creative conventions.
2. Write narration that hits the word budget.
3. Apply the phonetic acronym table from step 2.
4. Honour the tone from `CREATIVE-DIRECTION.md` (formal vs. casual, expert vs. educational, fast vs. measured).
5. Avoid filler phrases ("As you can see...", "Now let's look at...") — they waste seconds without adding information.

### Step 5 — Insert SSML markers

Use ElevenLabs' supported markers:

- **Scene boundary breaks:** `<break time="0.4s" />` at every scene boundary. This gives the audio production step a detectable silence to align on.
- **Mid-sentence pauses:** em-dashes (`—`) for natural breath points within a sentence. ElevenLabs respects punctuation timing.
- **Emphasis:** ElevenLabs has limited emphasis SSML support. For key words, rely on natural narration phrasing and the voice's default emphasis behaviour rather than explicit tags.

### Step 6 — Produce the timing table

Write `voiceover-timing-table.md`:

```markdown
| Seq | Name | Duration | Word budget | Actual word count | Tolerance | OK? |
|-----|------|----------|-------------|-------------------|-----------|-----|
| 01  | dashboard-intro | 4.0s | 10 | 11 | 9–11 | ✅ |
| 02  | edd-escalation-review | 8.0s | 20 | 24 | 18–22 | ❌ over |
| ... | | | | | | |

**Total runtime (script):** [N] s vs. **video duration:** [M] s — match status.
```

If any scene is outside tolerance, revise the narration for that scene only. Do not change scene durations to fit the script — that's the wrong direction.

### Step 7 — Produce the paste-ready ElevenLabs block

Write `voiceover-script.md` at the project root. Structure:

```markdown
# Voiceover Script — [Project name]

## ElevenLabs settings
- **Model:** eleven_multilingual_v2 (or specify alternative)
- **Voice:** [voice name — e.g. "Rachel" or a project-specific selection]
- **Stability:** 0.45 (default — adjust higher for measured walkthroughs, lower for marketing energy)
- **Style:** 0.10 (default — increase for more expressive delivery)
- **Speaker boost:** on
- **Output format:** mp3_44100_128

## Pronunciation guide
[The filled acronym table from step 2]

## Paste-ready script

```text
[Scene 1 narration with phonetic acronyms and <break> tags]
<break time="0.4s" />
[Scene 2 narration]
<break time="0.4s" />
[Scene 3 narration]
...
```

Copy the block above into ElevenLabs exactly as written.
```

### Step 8 — Validation pass

Before presenting to the human:

1. Read your own draft aloud (or use the text-length-to-time calculation): does the total length match the video duration within ±10%?
2. Are all acronyms phonetically hyphenated (or flagged as pronounced-as-word)?
3. Are there `<break time="0.4s" />` tags between every scene?
4. Does every key message from `brief-summary.md` appear in the script?
5. Does the tone match `CREATIVE-DIRECTION.md`?

### Step 9 — Human review

Present `voiceover-script.md` and the timing table:

> "Here's the script written to the locked scene durations. The paste-ready block is in `voiceover-script.md`. Want any phrasing changes before I send it to ElevenLabs?"

Iterate with the human on phrasing. The human's job here is voice/tone — yours is timing and structure. Defer to them on word choice.

## Outputs

- `voiceover-script.md` — full spec with paste-ready script, pronunciation guide, ElevenLabs settings
- `voiceover-timing-table.md` — per-scene timing validation
- Updated `resources/acronym-list.md` with phonetic column filled

## Tools used

`Read`, `Write`.

Note: `/hyperframes-media` includes a Kokoro TTS option, but for v1 we use ElevenLabs (human-assisted generation). The agent does not call any TTS tool directly in v1 — `audio-production` receives the generated MP3 from the human.

## Success criteria

- The proposed script, at the baseline rate, hits within ±10% of every scene's target duration.
- Every acronym in the script appears in `acronym-list.md` with its pronunciation rule.
- Every scene boundary has a `<break time="0.4s" />` tag (or equivalent SSML pause).
- Total script length matches video duration within ±10%.
- Every key message from `brief-summary.md` is represented in the script.
- Tone matches `CREATIVE-DIRECTION.md`.
- Human has reviewed and approved the script for ElevenLabs generation.
