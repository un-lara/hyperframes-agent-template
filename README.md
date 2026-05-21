# hyperframes-agent-template

Template repo for the Motion Design Agent — a Claude Code agent that owns the full HyperFrames video production lifecycle from brief intake to rendered MP4.

## Scaffold a new project

```bash
npx degit github.com/[your-org]/hyperframes-agent-template my-video-project
cd my-video-project
claude
```

Then in Claude Code, run `/project-init` to start the conversational kickoff.

## What's inside

```
.claude/commands/      ← Slash command skills (project-init, source-extraction, etc.)
CLAUDE.md              ← Agent persona, core rules, skill invocation map
PROJECT-STATE.json     ← Empty phase tracker, populated by project-init
resources/             ← Asset directory (screenshots, characters, brief, etc.)
```

When `/project-init` runs `npx hyperframes init`, the HyperFrames-generated `CLAUDE.md` is **merged** with the template `CLAUDE.md` — not overwritten. Critical framework rules from HyperFrames must be preserved. See `.claude/commands/project-init.md` step 10 for the merge procedure.

## Skill set

Six skills built so far (00–05). Skills 06–15 are planned — see the architecture docs at the parent project.

| Skill | Purpose |
|-------|---------|
| `/project-init` | Conversational kickoff, scaffold, CLAUDE.md merge |
| `/source-extraction` | Navigate URL/simulation source, screenshot, map flow |
| `/resource-intake` | Parse brief, extract acronyms, detect characters |
| `/creative-direction` | Classify video type, set duration, define conventions |
| `/design-system` | Tokens + motion rules + visual preview |
| `/scene-planner` | Named scenes with locked durations |

## Implementation gotchas

Read these before authoring any HyperFrames composition:

1. Use `window.__timelines`, never `window.__sceneTl`
2. Run `npm run check` after every composition change
3. Merge CLAUDE.md, never overwrite
4. Use Claude in Chrome for JS-rendered sources, not `web_fetch`
5. Inline `build-root.mjs` is the only correct assembly approach
6. `/website-to-hyperframes` is a different skill from `/source-extraction`

Full detail in `CLAUDE.md`.
