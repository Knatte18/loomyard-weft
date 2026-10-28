# Keep lym-wide rules out of CLAUDE.md; ship them in stencils and skills

`CLAUDE.md` mixed rules for developing LoomYard with rules that hold for every hub developed with lym.
A rule of the second kind written here is read by other hubs as a lym rule; the quarry hub's orch began taking in LoomYard's issues that way.
After this change `CLAUDE.md` holds only rules for developing LoomYard itself, and every rule for any use of lym lives in lym's code or in a stencil or skill lym ships.

## What changes

`CLAUDE.md` loses these rules:

- That agents lym spawns get the `PATTERN.md` overview inlined; the pattern directive stencils already carry it.
- That Batten's Land row runs the hub's `deploy` command after a landing into a `deploy_branches` branch; `internal/battenshed` enforces it and the orch role stencil states it.
- The "Notes go in git" details: worktree teardown with the memory store, `_lym/raddle/` regeneration, and where orch-only rules go.
- That the building task never changes its own entry's body or scope, that landing marks the entry done and prune removes it, and the run-lock rule; the board's run lock enforces the last two and the discussion, plan, darn and review stencils state them.
- That the board is read and written only through `lym board`; the board skill states it.

The "Notes go in git" section becomes "This file holds LoomYard development rules only".
It states the kind split, so a later edit does not put a lym-wide rule back.

Two rules had no shipped home and gain one:

- `contracts/stencils/orch/orch-template-role.md`, "Scratch and notes": never keep a durable note in an engine's session memory store.
- `plugins/ly/skills/board/SKILL.md`, "Writing a body": a body holds an entry's design while it is unbuilt, and the building task never rewrites its own entry's body or scope.

## Design decisions

- The `_lym/raddle/` regeneration sentence is dropped, not moved: no lym code regenerates `_lym/raddle/`, so it described no built behaviour.
- Rules for working in one's own worktree, the cgo build, `update-plugins.sh`, the `main` push rule and the other LoomYard conventions stay.
  "Work only in your own worktree" stays because it governs hand-started sessions in this repository, which no stencil reaches.
- The memory-store rule goes only into the orch stencil, the one lym role that keeps notes across sessions.
- The shipped stencils, specs and board skill were scanned for text naming LoomYard where it should name the hub's own repository; none was found, so no stencil text changes for that.
  The remaining mentions are lym's own bug reports and the code-review rubric's own-standard line.
- No new enforcement code.

## To check by hand

- The stencil edits ship with lym; the stencil tests in `contracts/stencils` were not run for this description.
- A session started by hand in another hub's worktree reads no stencil, so it gets the moved rules only when it loads the board skill.
