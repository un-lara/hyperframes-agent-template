# composition-builder

Assemble all scene compositions into a single inline root `index.html` with crossfaded transitions.

**The only correct assembly approach is the inline build via `build-root.mjs`.** Sub-composition switching via `data-composition-src` is broken in HyperFrames ≤0.5.7 — do not use it, do not offer it as an alternative.

## Inputs required

- All `compositions/seq[NN]-*.html` files (from `scene-author`)
- `SCENE-PLAN.md` (for start-offset calculation per scene)
- `design.md` (for crossfade timing and easing)
- `ANIMATION-RULES.md` (for the crossfade pattern)

## Required HyperFrames skill invocations

- **`/hyperframes`** — Required. Composition assembly patterns, root timeline construction, how individual scene timelines compose into a master timeline.
- **`/gsap`** — Required. Master timeline construction patterns; how to offset and chain sub-timelines.
- **`/hyperframes-cli`** — Required. `npx hyperframes inspect` for spot-checking the assembled `index.html`.

Invoke these before running the build.

## Steps

### Step 1 — Read all scene files

For each `compositions/seq[NN]-*.html`:
- Read the file
- Parse out the `<style>` block, the `<body>` contents, and the `<script>` block
- Extract the `data-scene` value (this is the scene's identifier)
- Verify the timeline is registered on `window.__timelines["..."]` — if any scene uses `window.__sceneTl`, halt and loop back to `scene-author` to fix the scene before continuing

### Step 2 — Generate or update `build-root.mjs`

Write `build-root.mjs` at the project root. The script reads each scene file, scopes styles and scripts to avoid collisions, inlines all scenes into a single `index.html`, and constructs the master GSAP timeline.

Structure of `build-root.mjs`:

```javascript
import { readFileSync, writeFileSync } from 'node:fs';
import { join } from 'node:path';

// 1. Read SCENE-PLAN.md to get the ordered list of confirmed scenes and their durations.
const scenes = parseScenePlan('./SCENE-PLAN.md');

// 2. For each scene file:
//    - Read compositions/seq[NN]-[name].html
//    - Extract <style>, <body> innerHTML, <script>
//    - Scope styles: wrap selectors with a scene-id prefix to prevent leakage
//    - Scope scripts: wrap in an IIFE or module scope, exposing only the timeline registration

// 3. Compute start offsets for each scene:
//    scene[0] starts at 0
//    scene[N] starts at sum(scenes[0..N-1].duration) - (crossfade / 2)
//    (the crossfade overlaps the seam by crossfade duration)

// 4. Build the master timeline. Each scene's timeline plays at its computed offset, with a 0.5s opacity crossfade at every seam (0.25s before / 0.25s after the seam).

// 5. Write the assembled output to ./index.html:
//    - Combined <style> (all scoped scene styles + global tokens from design.md)
//    - <body> with each scene's content wrapped in a <section data-scene="..."> for visibility control
//    - Master <script> that:
//        a. Constructs each scene's gsap.timeline (re-running the scene's script in scope)
//        b. Composes them into a paused master timeline
//        c. Adds crossfade tweens at each seam
//        d. Registers the master on window.__timelines["root"]

console.log('Assembled index.html with', scenes.length, 'scenes, total duration', totalDuration.toFixed(2), 's');
```

Use the actual structure of a working `build-root.mjs` from a previous project as the reference template if one exists (see `/Users/laraunsworth/Documents/Traverse/HyperFrames/PayPath Marketing Video/paypath-walkthrough/build-root.mjs` for the canonical example).

### Step 3 — Crossfade pattern

The crossfade at every scene seam follows the rule from `ANIMATION-RULES.md` (default 0.5s, possibly tighter for accurate-mode walkthroughs).

For each seam between scene N and scene N+1:
- Scene N's opacity goes from 1 → 0 over `crossfade/2` seconds, starting at `(scene N start + scene N duration) - (crossfade/2)`
- Scene N+1's opacity goes from 0 → 1 over `crossfade/2` seconds, starting at the same moment
- The total seam time is `crossfade` seconds (half before, half after the visual handoff point)

Both tweens use the easing from `ANIMATION-RULES.md` (typically `ease-default` for software/walkthroughs, `expo.out` for marketing).

### Step 4 — `window.__timelines` registration

Every scene's timeline must be present on `window.__timelines["seq[NN]-[name]"]` in the assembled `index.html`. The master root timeline is registered on `window.__timelines["root"]`. HyperFrames seeks the root timeline during render.

Verify with a grep after build:

```bash
grep -c 'window\.__timelines\[' index.html   # should equal N scenes + 1 (root)
grep -c 'window\.__sceneTl' index.html        # MUST equal 0
```

### Step 5 — Run the build

```bash
node build-root.mjs
```

The script should print a summary: scenes assembled, total duration, output path.

### Step 6 — Verify the assembled `index.html`

1. Confirm `index.html` exists at the project root.
2. File size sanity check: should be approximately the sum of all individual scene file sizes, ±20%. Significantly smaller usually means scenes were truncated; significantly larger usually means styles weren't scoped/deduplicated.
3. Open with `Read` to spot-check structure:
   - Single `<head>` with combined styles
   - Single `<body>` containing all scenes as `<section data-scene="...">` blocks
   - Single master `<script>` constructing the root timeline
4. Confirm no `window.__sceneTl` references anywhere in `index.html` or `build-root.mjs`.

### Step 7 — Run `npm run check` on the assembled output

```bash
npm run check
```

This runs HyperFrames' lint + validate + inspect against the assembled `index.html`. Fix any errors:
- Style conflicts across scenes → tighten the scoping in `build-root.mjs`
- Missing timeline registration → check the master script's IIFE/module wrapping
- Duration mismatch → verify the offset calculation in `build-root.mjs`

Do NOT mark this skill complete until `npm run check` passes.

### Step 8 — Spot-check with hyperframes inspect

Invoke `/hyperframes-cli` patterns and run an inspect at the project's first interesting offset (typically the start of scene 2 or 3 — somewhere past the opening transition):

```bash
npx hyperframes inspect --offset 5s
```

Verify the output frame is a live frame (file size >15 KB) showing the expected scene content. A file size of ~8.5 KB is a black frame, which usually means the master timeline failed to construct or the scene timeline wasn't registered.

## Outputs

- `build-root.mjs` — the inline assembly script
- `index.html` — the assembled root composition

## Tools used

`Write`, `Read`, `Bash` (`node build-root.mjs`, `npm run check`, `npx hyperframes inspect`).

HyperFrames slash commands: `/hyperframes`, `/gsap`, `/hyperframes-cli`.

## Success criteria

- `index.html` exists at the project root.
- File size is approximately the sum of all scene files ±20%.
- Master timeline duration matches the `SCENE-PLAN.md` confirmed total within ±5 s.
- `npm run check` passes with zero errors on the assembled `index.html`.
- `grep -c 'window\.__sceneTl' index.html` returns `0`.
- `grep -c 'window\.__timelines\[' index.html` returns `N+1` (N scenes + the root timeline).
- A spot-check `npx hyperframes inspect --offset [t]s` returns a live frame (>15 KB), not a black frame.
- `data-composition-src` is not used anywhere in `build-root.mjs` or `index.html`.
