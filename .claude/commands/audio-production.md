# audio-production

Receive (or generate) the voiceover, align it to the visual timeline, and mix in backing track. The output is a ready-to-render audio asset that lines up exactly with the locked scene timings.

**Conditional:** Only invoked if voiceover or backing audio is in the project's skill plan.

## Inputs required

- `assets/voiceover-v[N].mp3` — the generated VO (v1: human drops it after running ElevenLabs; v2: agent generates via API)
- `voiceover-script.md` and `voiceover-timing-table.md` (from `voiceover-script`)
- `SCENE-PLAN.md` (for scene boundary timing)
- `assets/backing-track.mp3` (optional — only if the brief calls for music)

## Required HyperFrames skill invocations

- **`/hyperframes-media`** — Optional but useful. Provides `transcribe` (Whisper) for word-level alignment if needed for fine adjustment, and `tts` (Kokoro) as an alternative TTS path. For v1 the agent does not use Kokoro — ElevenLabs is the voice — but the transcribe capability can help diagnose alignment issues.

## Steps

### Step 1 — Receive the VO file (v1) or generate it (v2)

**v1 (human-assisted):**

1. Confirm the human has dropped `assets/voiceover-v[N].mp3` into the project. The version number increments on every regeneration.
2. Validate the file exists and is a playable MP3: `file assets/voiceover-v[N].mp3` should report MP3 audio.

**v2 (API target):**

1. Call ElevenLabs API with the paste-ready script and settings from `voiceover-script.md`.
2. Save the result to `assets/voiceover-v[N].mp3`.
3. Continue from step 2 below.

### Step 2 — Detect silence gaps

Run ffmpeg's silence detection to locate the gaps between scenes (which correspond to the `<break time="0.4s" />` tags in the script):

```bash
ffmpeg -i assets/voiceover-v[N].mp3 -af "silencedetect=noise=-32dB:d=0.6" -f null - 2>&1 | grep silence_
```

This outputs a list of `silence_start` and `silence_end` timestamps. Capture them.

Adjust the noise floor (`-32dB`) and duration (`0.6s`) thresholds if detection is too sensitive or too lax for the specific voice. ElevenLabs default voices typically work at `-32dB / 0.6s`.

### Step 3 — Map silence gaps to scene boundaries

From `voiceover-timing-table.md`, you know how many scene boundaries to expect: `N-1` boundaries for `N` scenes.

Count detected gaps and compare to expected boundary count.

- **Gap count matches** → proceed to alignment mapping.
- **Gaps missing** → likely the TTS rendered some scene boundaries as continuous speech. Either the script needs a longer `<break>` and a re-generation, or the alignment will need manual gap insertion.
- **Extra gaps** → likely the VO has natural pauses within scenes that the detector picked up. Use the longest detected gaps as the scene-boundary candidates and discard shorter ones.

### Step 4 — Produce the alignment report

Write `audio/alignment-report.md`:

```markdown
# Alignment Report — VO v[N]

| Scene boundary | Expected (video) | Detected (audio) | Drift | Status |
|----------------|------------------|------------------|-------|--------|
| After seq01 | 4.000s | 4.120s | +0.12s | ✅ within tolerance |
| After seq02 | 12.000s | 11.350s | -0.65s | ⚠️ slight |
| After seq03 | 18.500s | 18.480s | -0.02s | ✅ |
| ... | | | | |

**Verdict:** Tolerance = ±1.0s per boundary. [N] of [M] boundaries within tolerance.
```

### Step 5 — Align the VO to the visual timeline

If alignment is within ±1s per boundary:

Use ffmpeg `atrim` (trim) and `apad` (pad with silence) to nudge each scene's audio segment to match its expected start. Produce `assets/voiceover-aligned.mp3`:

```bash
# Example: extract each scene's audio segment and concatenate with silence padding to scene boundary
ffmpeg -i assets/voiceover-v[N].mp3 -af "atrim=start=0:end=4.0,apad=pad_dur=0.0" segment-01.mp3
ffmpeg -i assets/voiceover-v[N].mp3 -af "atrim=start=4.12:end=11.35,apad=pad_dur=0.77" segment-02.mp3
# ... concat all segments
ffmpeg -i "concat:segment-01.mp3|segment-02.mp3|..." -c copy assets/voiceover-aligned.mp3
```

In practice, write a small alignment script (`audio/align.mjs` or similar) that reads the alignment report and computes the trim/pad operations programmatically.

If alignment is NOT within tolerance (drift > 1s on any boundary): flag to the human and queue a VO regeneration with a script revision (likely the offending scene's narration is too long or too short for the budget).

### Step 6 — Generate the waveform cache

HyperFrames uses a waveform cache for audio-reactive features and inspection. Generate it:

```bash
mkdir -p .waveform-cache
# Either use HyperFrames' built-in waveform generation or ffmpeg + a small script
npx hyperframes media waveform assets/voiceover-aligned.mp3 --output .waveform-cache/voiceover-aligned.json
```

(Adjust the exact command per the `/hyperframes-media` skill specification.)

### Step 7 — Backing track loop (if applicable)

If `assets/backing-track.mp3` exists:

1. Validate the loop seam: play the file end and the beginning back-to-back. If there's an audible click or sudden change, the track isn't loop-safe — flag to the human or apply a 0.3s crossfade at the seam.
2. Loop the backing track to match the total video duration:

   ```bash
   # Get total video duration from SCENE-PLAN.md
   ffmpeg -stream_loop -1 -i assets/backing-track.mp3 -t [video_duration]s -c copy assets/backing-track-loop.mp3
   ```

3. Lower the volume to 15–25% of the VO level (per `CREATIVE-DIRECTION.md` audio convention for software/walkthrough types) using `volume` filter:

   ```bash
   ffmpeg -i assets/backing-track-loop.mp3 -af "volume=0.20" assets/backing-track-mixed.mp3
   ```

4. Use `volumedetect` to confirm levels are sensible (peak around -18 dB for the backing track, peak around -3 dB for VO):

   ```bash
   ffmpeg -i assets/backing-track-mixed.mp3 -af "volumedetect" -f null - 2>&1 | grep mean_volume
   ```

### Step 8 — Human listens to the aligned VO and approves

Present the aligned VO and (if applicable) the backing track:

> "Voiceover aligned — all [N-1] scene boundaries within ±1s of plan. Listen to `assets/voiceover-aligned.mp3` (and `assets/backing-track-mixed.mp3` if applicable). Approve or flag anything you want regenerated."

If the human rejects: queue a new ElevenLabs run with whatever script revisions they specify, re-enter this skill from step 1 with v[N+1].

## Outputs

- `assets/voiceover-aligned.mp3` — the VO trimmed/padded to scene boundaries
- `assets/backing-track-loop.mp3` and/or `backing-track-mixed.mp3` — if backing track supplied
- `audio/alignment-report.md` — per-boundary drift report
- `.waveform-cache/voiceover-aligned.json` — waveform cache for HyperFrames

## Tools used

`Bash` (ffmpeg: `silencedetect`, `atrim`, `apad`, `concat`, `stream_loop`, `volume`, `volumedetect`), `Read`, `Write`.

HyperFrames slash commands: `/hyperframes-media` for waveform generation and (optionally) Whisper transcription for alignment debugging.

## Success criteria

- Human has listened to and approved the aligned VO.
- All scene boundary alignment gaps are within ±1s per `alignment-report.md`.
- `assets/voiceover-aligned.mp3` exists and matches the total video duration within ±0.5s.
- Waveform cache exists at `.waveform-cache/voiceover-aligned.json`.
- If backing track applies: it loops cleanly to the video duration, is mixed at 15–25% VO volume per the type-specific convention, and human has approved the mix.
