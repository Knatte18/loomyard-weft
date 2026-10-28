# Move lym-wide rules out of CLAUDE.md into stencils and skills lym ships

`CLAUDE.md` is read only by sessions working in this repository, yet it held rules about how lym behaves in every hub.
Read as lym rules, LoomYard-only rules leak into other hubs, as when the quarry hub's orch took in LoomYard's issues.
This change leaves `CLAUDE.md` with only the rules for developing LoomYard, and moves each lym-wide rule into the code, stencil or skill lym ships.

## What changes

- `CLAUDE.md` drops the lym-wide rules:
  that spawned agents get the `PATTERN.md` overview inlined,
  how Batten's Land row runs the hub's `deploy` command,
  that task worktrees are torn down with the memory store,
  how `_lym/raddle/` reaches the parent,
  where orch-only rules go,
  that an unbuilt design lives in its board entry's body and the building task never rewrites it,
  the run-lock rule,
  and that the board is read and written only through `lym board`.
- The "Notes go in git" section becomes "This file holds LoomYard development rules only".
  It states the split itself: a rule for developing LoomYard goes in `CLAUDE.md` or a code comment, and a rule for every use of lym goes into lym's code or its shipped stencils and skills, never into `CLAUDE.md` or a package's `doc.go`.
- The orch role stencil's "Scratch and notes" section now forbids keeping a durable note in an engine's session memory store.
- The board skill's "Writing a body" section now states that a body holds an entry's design while it is unbuilt, and that the building task never rewrites its own entry's body or scope.

## Design decisions

- Every other removed rule already had a home that ships with lym:
  `internal/battenshed` runs the deploy and the orch stencil describes it,
  the board's run lock and the discussion, plan, darn and review stencils carry the run-lock rule,
  the orch stencil names the orch-rules files,
  and the board skill requires `lym board`.
- The `_lym/raddle/` sentence is dropped with no new home, since no lym code regenerates `_lym/raddle/`.
- "Work only in your own worktree" stays in `CLAUDE.md`.
  It governs sessions that develop this repository by hand, and no lym stencil reaches those sessions.
- The memory-store rule goes to the orch only, because it is the lym role that keeps notes across sessions.
  The shared edit directive states only how a role changes files, so the rule does not belong there.
- The stencil scan found no rule naming LoomYard where it should name the hub's repository:
  each mention is about lym's own bugs or friction, or the code-review rubric's own-standard line.
