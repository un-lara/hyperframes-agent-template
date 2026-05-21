# render

Trigger the HyperFrames render and manage the render → audio QA → human watch → polish loop.

## Inputs required

- `index.html` (from `composition-builder`, snapshot-QA-approved)
- `assets/voiceover-aligned.mp3` (from `audio-production`, if voiceover applies)
- `assets/backing-track-mixed.mp3` (from `audio-production`, if backing track applies)
- `SCENE-PLAN.md` (for duration verification)

## Required HyperFrames skill invocations

- **`/hyperframes-cli`** — Required. `npx hyperframes render` and `npx hyperframes inspect` patterns and how to interpret render meta sidecars.
- **`/hyperframes`** — Useful. How audio elements are referenced in the composition and how the renderer handles them.

## Steps

### Step 1 — Pre-flight checks

Before triggering the render, confirm:

1. `index.html` exists and `snapshot-qa` has passed for this version.
2. `assets/voiceover-aligned.mp3` exists (if VO applies).
3. `assets/backing-track-mixed.mp3` exists (if backing track applies).
4. `index.html` references the audio files correctly. Spot-check the `<audio>` elements:

   ```bash
   grep -o '<audio[^>]*>' index.html
   ```

   Every `<audio>` should point to an existing file.

5. Disk space — renders can be 50–200 MB depending on duration. `df -h .` should show comfortable headroom.

If any pre-flight fails: halt and resolve before rendering.

### Step 2 — Determine version number

Read `renders/render-log.md` (create if it doesn't exist). The next render is `v[N+1]` where `N` is the highest version in the log. If first render: `v1`.

### Step 3 — Trigger the render

```bash
npx hyperframes render --output renders/[project]-v[N].mp4
```

This typically takes 30 s to 5 min depending on duration and complexity. Stream the output and watch for errors.

When complete, the renderer also writes a meta sidecar (typically `renders/[project]-v[N].mp4.meta.json` or similar — check `/hyperframes-cli`).

### Step 4 — Check the render meta sidecar

Read the sidecar:

```bash
cat renders/[project]-v[N].mp4.meta.json
```

Look for:
- `status: "success"` — proceed
- `status: "failed"` — read the error. Common causes: missing asset, timeline registration error, audio reference broken. Diagnose, fix, retry up to 2 times. After 2 failures, halt and escalate to the human.
- `warnings:` — review; some are informational, some matter.

### Step 5 — Spot-check the rendered output

Quick technical checks on the MP4 before handing off:

```bash
# Duration check
ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 renders/[project]-v[N].mp4
# Should be within ±2s of SCENE-PLAN.md total duration

# Audio stream present (if VO applies)
ffprobe -v error -select_streams a -show_entries stream=codec_type renders/[project]-v[N].mp4
# Should output "codec_type=audio"

# Video stream healthy
ffprobe -v error -select_streams v -show_entries stream=width,height,r_frame_rate renders/[project]-v[N].mp4
# Width/height should match HyperFrames project config (typically 1920x1080), framerate typically 30 or 60
```

### Step 6 — Pass to audio QA immediately

Do NOT show the render to the human yet. First, invoke `/audio-qa`. The render only reaches the human after `audio-qa` confirms VO content matches visual content at every scene boundary.

Loop:
- `audio-qa` passes → continue to step 7.
- `audio-qa` fails → diagnose the cause:
  - VO timing drift → loop back to `audio-production` for re-alignment with a tighter tolerance.
  - VO content mismatch (wrong scene's narration playing during a scene) → loop back to `voiceover-script` for a script revision (likely a misordered or duplicated segment), then `audio-production`, then re-render.

### Step 7 — Human watches and gives polish direction

Once audio QA passes, present the render to the human:

> "Render v[N] is ready and audio QA passed. Watch `renders/[project]-v[N].mp4` and let me know: approve to ship, or what to polish?"

The human's feedback typically falls into categories:

- **Scene-level visual polish** (e.g. "the table animation in seq04 feels slow") → loop back to `scene-author` for that specific scene → `composition-builder` → `snapshot-qa` → re-render (v[N+1]).
- **Pacing change** (e.g. "scene 6 needs more breathing room") → update `SCENE-PLAN.md` → re-run `voiceover-script` → `audio-production` → `scene-author` (only if scene timing changed) → `composition-builder` → re-render.
- **Copy or script change** (e.g. "the VO in scene 3 is awkward") → loop back to `voiceover-script` → `audio-production` → re-render.
- **Ship it** → proceed to `post-production`.

### Step 8 — Log this version in `renders/render-log.md`

Append an entry:

```markdown
## v[N] — [date]
- Render path: `renders/[project]-v[N].mp4`
- Duration: [actual]s (planned [planned]s)
- File size: [size]
- Audio QA: ✅ passed | ❌ failed and looped to [skill]
- Human verdict: ✅ ship | ⏭️ polish: [summary of changes requested]
- Polish actions: [if any — list of scenes / skills re-run]
```

## Outputs

- `renders/[project]-v[N].mp4` — the rendered video
- `renders/[project]-v[N].mp4.meta.json` — the render meta sidecar (from the CLI)
- `renders/render-log.md` — versioned history with verdicts and polish notes

## Tools used

`Bash` (`npx hyperframes render`, `npx hyperframes inspect`, `ffprobe`), `Read`, `Write`.

HyperFrames slash commands: `/hyperframes-cli`, `/hyperframes`.

## Success criteria

- A successful render exists in `renders/`.
- Render meta sidecar reports `status: "success"`.
- Duration matches `SCENE-PLAN.md` total within ±2s.
- Audio QA has passed for this render before the human saw it.
- Human has watched and either approved (→ `post-production`) or specified polish direction.
- If polish was requested, the appropriate skill loop ran and produced a new version that was re-checked.
- Every version of every render is logged in `renders/render-log.md`.
