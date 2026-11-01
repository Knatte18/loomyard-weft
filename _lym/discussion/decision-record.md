## Goal

```markdown
## Seen (orch, 2026-11-01)

The batching sentence every shuttle-launched Claude session gets ("Request every call whose arguments you already know in one message, …", `batchingSentence` in `internal/shuttleengine/claudeengine/settings.go`) sits at the end of Claude Code's long system prompt, which already asks for parallel tool calls itself.
After `turn-batching` landed, Webster forks went from 1.37 to about 1.7 read, grep and edit calls a turn (`tools/forkaudit`, 27 forks of `warp-visibility`, `interactive-recipe` and `run-verbs-shared`).
Claude Code puts a project's CLAUDE files into the conversation as a context block under "These instructions OVERRIDE any default behavior and you MUST follow them exactly as written", a stronger frame than an appended system-prompt sentence; another user reported cutting token use by putting such an instruction there.

## Idea (operator, 2026-11-01)

Test it with heddle, with no code change: a heddle entry aimed at the `claude-file` target puts the batching instruction into the rendered `CLAUDE.local.md` at every worktree's root, which every Claude session in the hub reads, Webster's forks through Merriam's context included.
After two or three tapestry runs, `tools/forkaudit` measures calls per turn against the 1.7 above.
The test is best run after note `edit-with-edit-tool` has landed, or measured on read and search calls alone, so that the edit change does not confound it.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-01): The trial moves into code: lym's instructions reach the CLAUDE-file context from claudeengine

Decision: Option (b). claudeengine delivers the batching sentence through the CLAUDE-file context every shuttle-launched Claude session reads, the frame Claude Code marks as overriding defaults, instead of the appended system prompt, with no heddle entry. Build it as a channel lym owns for its own agent instructions, so plan-edits-before-gate, which waits on this task, adds its instruction through the same channel. Read how heddle renders CLAUDE.local.md and pick the form that keeps lym's text apart from the repository's heddle entries; the sentence leaves the appended system prompt in the same change, so a session never gets it twice.

Rationale: PATTERN-lym-instructions-in-code, landed after the entry was written, keeps lym's own instructions in lym's code and heddle for a repository's own; the trial's question, whether the CLAUDE-file frame moves agents more than the system prompt, survives the move. The effect is measured for the whole project by note agent-turns-effect.

### Added after Discussion (parent, 2026-11-01): lym's instructions land in the rendered CLAUDE.local.md

Decision: The channel writes lym's own agent instructions into the rendered CLAUDE.local.md at each worktree root, as a section of its own that lym's code supplies, apart from the repository's heddle entries; not into any other context. The batching sentence is its first content and plan-edits-before-gate adds the second.

Rationale: Operator, 2026-11-01: CLAUDE.local.md is where these instructions belong; heddle is per repository, so lym's text comes from lym's code and only shares the rendered file.

### Added after Discussion (parent, 2026-11-01): The channel is a shipped layer of heddle, replacing the claudeengine channel

Decision: This replaces the two earlier parent decisions' channel. Heddle gains a second layer: a store lym ships inside its binary (embedded in internal/heddleengine, in the same entry format as heddle.json), read-only, changed only by a lym release. Rendering, show and passages read the shipped layer and the repository's heddle.json together for every target: the shipped entries render first, as a section of their own, the repository's after. lym heddle set and remove refuse a shipped slug, and lym heddle list marks each entry's layer. The batching sentence becomes the shipped layer's first entry, aimed at the claude-file target, and leaves claudeengine's appended system prompt in the same change, so a session never gets it twice. plan-edits-before-gate adds the second shipped entry. heddle-orch-target, in review now, also changes internal/heddleengine; take its target in when it lands.

Rationale: Operator, 2026-11-01: heddle is per repository, and lym's own instructions belong in the same rendered files; a read-only layer shipped with lym keeps them in lym's code (PATTERN-lym-instructions-in-code) and reuses heddle's targets and rendering instead of a second mechanism. internal/heddleengine is about two thousand lines with its tests, so the layer fits this task.

### Added after Discussion (parent, 2026-11-01): A hub can override the shipped heddle layer for tuning

Decision: A hub setting, heddle.lym_override, off by default, lets lym heddle set --layer lym store an override of a shipped entry's text in the Bolt checkout beside heddle.json, and lym heddle remove --layer lym drop it. With the setting on, rendering, show and passages use an overridden entry's text in place of the shipped one, with no build or commit; with it off, overrides are ignored and set --layer lym is refused. lym heddle list marks each overridden entry. A setting outside an override still refuses set and remove on a shipped slug. The loomyard hub turns the setting on; once a text proves itself, a task copies it into the shipped layer and the override is removed.

Rationale: Operator, 2026-11-01: in loomyard lym's own instructions are tuned and benchmarked quickly, and a new binary per change is too slow; the override keeps the shipped layer read-only everywhere else.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
