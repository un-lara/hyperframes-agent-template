# audio-qa

Scan the rendered video at every scene boundary to verify that audio content aligns with visual content. Catches misalignment before the human watches.

This skill runs **automatically** after `render` completes. The human does not see the render until `audio-qa` passes.

## Inputs required

- `renders/[project]-v[N].mp4` (from `render`)
- `SCENE-PLAN.md` (scene names, start/end times)
- `voiceover-script.md` (what the VO is expected to say in each scene)

## Required HyperFrames skill invocations

- **`/hyperframes-media`** — Useful. Whisper transcription via the `transcribe` tool produces word-level transcripts that make audio/script comparison much sharper than ear-listening.

## Steps

### Step 1 — Extract audio segments at scene boundaries

For each scene boundary in `SCENE-PLAN.md`, extract a short audio clip (typically 3 seconds: 1.5 s before the boundary, 1.5 s after). Use ffmpeg:

```bash
ffmpeg -i renders/[project]-v[N].mp4 -ss [scene_start - 1.5]s -t 3 -vn -acodec copy audio/qa-clips/boundary-[NN].mp3
```

Repeat for every scene boundary (N-1 boundaries for N scenes), plus one extract at scene 1 start (0s) and one at the final scene end.

### Step 2 — Transcribe each clip

Use `/hyperframes-media`'s Whisper transcription on each clip:

```bash
npx hyperframes media transcribe audio/qa-clips/boundary-[NN].mp3 --output audio/qa-transcripts/boundary-[NN].json
```

The output is word-level transcript JSON with timestamps.

If Whisper transcription isn't available or fails: fall back to listening with multimodal audio understanding — read the clip and describe what's said. Less precise but workable.

### Step 3 — Compare audio content to expected script content per scene

For each scene boundary, you now have:

- The actual audio words spoken at the boundary (from transcription)
- The expected script content per `voiceover-script.md` for the scenes on either side of the boundary

Compare:

- **At a boundary "after seq02 / before seq03":** the last words of seq02's script segment should be heard just before the boundary, and the first words of seq03's script segment just after.
- **Mismatch patterns to detect:**
  - The pre-boundary audio matches a *different* scene's expected content → alignment drift; one or more scenes are offset.
  - The post-boundary audio is silence or matches the previous scene's content → VO over-runs the scene boundary.
  - Audio content matches a scene's script but the scene shown at that offset is different → visual/audio swap (rare, but happens with composition-builder mistakes).
  - No audio at all where audio is expected → audio reference broken in `index.html`.

### Step 4 — Produce `audio/audio-qa-report.md`

```markdown
# Audio QA Report — Render v[N]

| Boundary | Expected pre (scene end) | Heard pre | Expected post (scene start) | Heard post | Verdict |
|----------|--------------------------|-----------|----------------------------|------------|---------|
| 01→02 | "...verifying compliance status." | "...verifying compliance status." | "Next, E-D-D escalation..." | "Next, E-D-D escalation..." | ✅ aligned |
| 02→03 | "...the case is escalated." | "...the case is" | "Marcus joins the review..." | "escalated. Marcus joins..." | ⚠️ +0.4s drift on seq02 |
| 03→04 | ... | ... | ... | ... | ❌ wrong scene content |

**Summary:** [N] of [M] boundaries aligned. [P] critical failures.
```

### Step 5 — Diagnose and route failures

For any boundary that fails:

- **Small drift (< 1.0s)** → loop back to `audio-production` for re-alignment with the corrected gap mapping.
- **Wrong content at boundary** → likely a script ordering issue. Loop back to `voiceover-script` to check the scene order in the script matches `SCENE-PLAN.md`. Then `audio-production` → re-render.
- **Silence where audio expected** → likely a missing `<audio>` reference in `index.html`. Loop back to `composition-builder` to verify audio inclusion.
- **Audio extends beyond a scene** → the scene's narration is longer than the scene allows. Either trim the narration (loop to `voiceover-script`) or extend the scene (loop to `scene-planner` → cascade).

### Step 6 — Auto-route or auto-clear

- **All boundaries pass** → flag the render as audio-cleared. Notify `render` to present to the human.
- **Any boundary fails** → diagnose, propose the minimum fix, route to the appropriate skill. Do NOT present the render to the human until audio QA passes.

This is the one skill where the agent absolutely does not loop in the human until either pass or repeated failure. The human's time is too expensive to spend on misaligned renders.

### Step 7 — Edge case: no VO

If the project is silent/music-only (no `voiceover-aligned.mp3`):

- Skip transcription steps.
- Verify only that the backing track plays for the full video duration without dropouts.
- Use `ffprobe` to check audio stream continuity:

  ```bash
  ffprobe -v error -show_entries packet=pts_time,size -select_streams a renders/[project]-v[N].mp4 | head -50
  ```

- A `audio-qa-report.md` is still written, with a section noting "Music-only — no VO alignment check needed; audio stream continuity verified".

## Outputs

- `audio/qa-clips/boundary-[NN].mp3` — extracted clips per boundary
- `audio/qa-transcripts/boundary-[NN].json` — Whisper transcripts per clip
- `audio/audio-qa-report.md` — per-boundary pass/fail with diagnosis

## Tools used

`Bash` (ffmpeg extraction, ffprobe), `Read`, `Write`.

HyperFrames slash commands: `/hyperframes-media` (transcribe via Whisper).

## Success criteria (auto-gate)

- All scene boundaries pass audio QA (audio content matches expected script content within timing tolerance).
- `audio-qa-report.md` shows all pass.
- Any failures have been automatically diagnosed and routed to the appropriate fix skill — the human is not shown the render until pass.
- For music-only videos: audio stream continuity is verified across the full duration.
