# snapshot-qa

Technically validate the composition by inspecting frames at key offsets. This is a **technical check** — not a creative one. The question is: are the frames rendering at all, and is the right scene showing at the right offset?

Creative critique of scene content happens earlier (in `creative-qa`) and later (in human review after render). Snapshot QA exists to catch broken composition assembly before rendering wastes minutes per attempt.

## Inputs required

- `index.html` (from `composition-builder`)
- `SCENE-PLAN.md` (for expected scene timing — what should be on screen at each offset)
- `design.md` (for expected visual baseline — what colours, what typography)

## Required HyperFrames skill invocations

- **`/hyperframes-cli`** — Required. The `npx hyperframes inspect --offset` command and how to interpret its output.

## Steps

### Step 1 — Calculate inspection offsets

Compute a representative set of timeline offsets to inspect. For each scene in `SCENE-PLAN.md`:

- **Opening frame:** `0.5s` (after the first crossfade resolves)
- **Scene midpoint:** for scene N, offset = `scene_start + (scene_duration / 2)`
- **Pre-seam frame:** `1s` before each scene's end (catches the scene before the crossfade starts)
- **Final frame:** `total_duration - 0.5s`

For a 90-second video with 10 scenes, this produces roughly 21 inspection offsets. Adjust upward for longer videos.

### Step 2 — Run inspect at each offset

For each calculated offset:

```bash
npx hyperframes inspect --offset [t]s
```

This produces a PNG in `snapshots/` named with the offset. The CLI typically also writes a small metadata sidecar.

Batch the calls — do not invoke them one-at-a-time conversationally. Run them in a loop or with parallel bash calls.

### Step 3 — Evaluate each frame

For each generated snapshot:

| Check | Pass | Fail |
|-------|------|------|
| File size | > 15 KB (a live, rendered frame) | ≈ 8.5 KB (a black frame — composition failed at this offset) |
| Scene match | The scene shown matches what `SCENE-PLAN.md` says should be at this offset | Wrong scene visible (timing drift) |
| Visual integrity | Design tokens applied, text legible, no broken layouts | Fallback colours visible, missing fonts, layout collapse |

For the visual integrity check, use `Read` on each snapshot PNG (multimodal vision) to confirm the scene looks correct against the expected scene description in `SCENE-PLAN.md`. This catches subtle issues that file-size alone misses — e.g. a scene that rendered but is missing its background image.

### Step 4 — Produce `snapshots/qa-report.md`

For each offset, record:

```markdown
| Offset | Expected scene | File size | Verdict | Notes |
|--------|---------------|-----------|---------|-------|
| 0.5s   | seq01-dashboard-intro | 47 KB | ✅ pass | Clean opening frame |
| 3.0s   | seq02-edd-escalation | 8.5 KB | ❌ fail | Black frame — likely seq02 timeline registration broken |
| 5.5s   | seq02-edd-escalation | 51 KB | ✅ pass | Mid-scene, table visible |
| ...    | | | | |

**Summary:** [N] of [M] offsets passed.
```

Include a summary of any failures and a diagnosis of likely cause.

### Step 5 — Diagnose failures and route fixes

If any frame fails:

- **Black frame at scene start** → likely the scene's timeline isn't registered correctly in `index.html`. Check `window.__timelines["seq[NN]-..."]` exists. Loop back to `composition-builder` to re-verify scoping.
- **Black frame in scene middle** → likely the scene's GSAP animations exceed the scene duration or there's a `display: none` toggled mid-scene by accident. Loop back to `scene-author` for that specific scene to fix the timing.
- **Wrong scene at offset** → master timeline offset miscalculation. Loop back to `composition-builder`; check the start-offset math.
- **Missing fonts / colours** → design tokens not propagated to assembled `index.html`. Loop back to `composition-builder` to verify the styles-scoping step preserved `:root` token definitions.

Re-author or re-build the affected file ONLY (do not re-author all scenes). Re-run `npm run check`, then re-run snapshot QA. Continue until all frames pass.

### Step 6 — Pass the snapshot grid to the human

Once all frames pass technically, present a snapshot grid (or list of paths) to the human for a quick visual review:

> "Snapshot QA passed all [N] offsets. Quick visual sanity check — here are the snapshots in `snapshots/`. Anything look off before I proceed to voiceover?"

The human's review here is fast — they're looking for "yes, this looks like the planned video" not detailed critique. Detailed critique happens after render.

## Outputs

- `snapshots/[offset]s.png` — one PNG per inspection offset
- `snapshots/qa-report.md` — pass/fail per offset with diagnosis

## Tools used

`Bash` (`npx hyperframes inspect`), `Read` (multimodal, for visual integrity check), `Write`.

HyperFrames slash commands: `/hyperframes-cli`.

## Success criteria

- All inspection frames are > 15 KB (no black frames).
- All scenes appear at correct offsets per `SCENE-PLAN.md` — no timing drift.
- Visual integrity check confirms design tokens applied and layouts intact at each offset.
- Human has reviewed the snapshot grid and approved (a quick "looks right" check).
- If failures occurred, they have been fixed and re-verified, not deferred.
