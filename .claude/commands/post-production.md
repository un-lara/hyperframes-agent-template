# post-production

Generate the two standard output variants and wrap the project. The goal is a clean, singular output — not an archive of permutations.

## Inputs required

- The approved render file from `renders/render-log.md` (latest version marked `ship`)
- `SCENE-PLAN.md` (for thumbnail offset selection and summary)
- `CREATIVE-DIRECTION.md` (for summary context)
- `PROJECT-STATE.json` (for project metadata)

## Steps

### Step 1 — Identify the approved render

Read `renders/render-log.md` and find the version the human marked `ship`. That's the input file for all variants. Call it `renders/[project]-v[N].mp4`.

If no version is marked `ship`: halt and ask the human which version to use. Do not pick yourself.

### Step 2 — Generate the normal-speed variant

This is just a clean copy (or re-mux) of the approved render at 1× speed:

```bash
ffmpeg -i renders/[project]-v[N].mp4 -c copy renders/[project]-v[N]-normal.mp4
```

Using `-c copy` avoids re-encoding — the file is bit-identical to the source for video and audio streams, just with a clarifying filename.

### Step 3 — Generate the 1.2× speed variant

Speed up both video and audio by 1.2×:

```bash
ffmpeg -i renders/[project]-v[N].mp4 \
  -filter_complex "[0:v]setpts=PTS/1.2[v];[0:a]atempo=1.2[a]" \
  -map "[v]" -map "[a]" \
  renders/[project]-v[N]-1.2x.mp4
```

`atempo=1.2` preserves pitch (audio doesn't get chipmunky); `setpts=PTS/1.2` speeds the video proportionally.

Verify both variants:

```bash
ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 renders/[project]-v[N]-normal.mp4
ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 renders/[project]-v[N]-1.2x.mp4
# Normal should match original duration; 1.2× should be original / 1.2
```

### Step 4 — Human selects ship version

> "Both variants generated. Which one ships? (Or both — that's fine if the use case requires it.)"

The human picks one or both. Their choice goes in `deliverables.json` as `role: "ship"`.

### Step 5 — Extract poster thumbnail

Pull a single frame from the ship version at 2.5s in (typically past the opening crossfade, into the first scene's settled state):

```bash
ffmpeg -ss 2.5 -i renders/[project]-v[N]-[ship-variant].mp4 -frames:v 1 -q:v 2 renders/[project]-thumbnail.png
```

If the scene at 2.5s isn't visually representative (e.g. it's a transitional moment), pick a more representative offset — typically the midpoint of the first content-heavy scene from `SCENE-PLAN.md`. Use multimodal vision to confirm the thumbnail looks like a poster frame for the video.

### Step 6 — Write `renders/deliverables.json`

```json
{
  "project": "[project-name]",
  "render_version": "v[N]",
  "approved_date": "[YYYY-MM-DD]",
  "deliverables": [
    {
      "filename": "[project]-v[N]-normal.mp4",
      "speed": "1.0x",
      "duration_seconds": [N],
      "file_size_bytes": [size],
      "role": "ship" | "alternate"
    },
    {
      "filename": "[project]-v[N]-1.2x.mp4",
      "speed": "1.2x",
      "duration_seconds": [N],
      "file_size_bytes": [size],
      "role": "ship" | "alternate"
    },
    {
      "filename": "[project]-thumbnail.png",
      "type": "poster",
      "extracted_from": "[ship variant filename]",
      "offset_seconds": 2.5
    }
  ]
}
```

Exactly one variant has `role: "ship"` (or both if the human selected both — in that case `role: "ship"` on both, with a note explaining the use cases).

### Step 7 — Write `project-summary.md`

The project-closing document. Structure:

```markdown
# Project Summary — [project name]

**Project type:** [from CREATIVE-DIRECTION.md]
**Ship version:** v[N] ([speed variant])
**Approved:** [date]
**Total runtime:** [N]s

## What was built
[2–3 sentences describing the video at a high level — what it shows, what message it carries, what it's for.]

## Scenes built
[List of confirmed scenes from SCENE-PLAN.md with final durations.]

## Scenes cut and why
[Any scenes marked `optional` that did not ship, with the reason — e.g. "seq07 character intro cut: brief didn't have enough character development to justify the screen time".]

## Branding applied
[Traverse standard | Custom from {source}] — see `design.md` for full token reference.

## Skills invoked
[List of all skills that ran, in order. Useful for understanding which conditional skills fired and which were skipped.]

## Total iterations
- Render versions: [N]
- Polish loops: [count]
- Audio QA loops: [count]

## Key decisions
[Notable creative or technical decisions made during production. E.g. "Switched from accurate to stylised mode mid-source-extraction because the simulation included too many irrelevant states", "Cut the character intro scene because it interrupted the narrative flow".]

## Production time
[Total time from project-init to ship, if tracked.]

## Final deliverables
See `renders/deliverables.json` for the file manifest.
```

### Step 8 — Update `PROJECT-STATE.json`

Mark the project complete:

```json
{
  "current_phase": "complete",
  "ship_version": "v[N]",
  "ship_date": "[YYYY-MM-DD]",
  "ship_files": ["..."]
}
```

## Outputs

- `renders/[project]-v[N]-normal.mp4` — 1× variant
- `renders/[project]-v[N]-1.2x.mp4` — 1.2× variant
- `renders/[project]-thumbnail.png` — poster frame
- `renders/deliverables.json` — file manifest with ship selection
- `project-summary.md` — project-closing summary
- Updated `PROJECT-STATE.json`

## Tools used

`Bash` (ffmpeg: speed adjustment, thumbnail extraction; ffprobe: duration verification), `Read` (multimodal — for thumbnail vision check), `Write`.

## Success criteria

- Both speed variants exist and are playable.
- Normal variant duration matches the source render.
- 1.2× variant duration equals source / 1.2 within ±0.2s.
- Thumbnail PNG exists and visually represents the video (verified via vision).
- `renders/deliverables.json` exists and lists exactly one (or both, if the human chose both) `ship` entry.
- `project-summary.md` exists and is complete.
- `PROJECT-STATE.json` shows `current_phase: "complete"`.
- **No additional variants generated.** No silent, no-outro, vertical, or other speed permutations unless the human explicitly requested them. The agent does not produce variants speculatively.
