# Card 13 — orch-docs

**Create:**
- `internal/orchengine/doc.go`

**Prosa:**
- `docs/overview.md`
- `CONSTRAINTS.md`
- `manifest/roadmap.md`

**Uses:**
- `internal/shuttleengine/doc.go`
- `internal/orchengine/watcher.go`
- `internal/orchcli/start.go`

**Intent:**
Land the module's documentation, per the repo's "docs land in the same task" rule.

`internal/orchengine/doc.go` is the module doc, as `internal/shuttleengine`'s package documentation is shuttle's.
It covers:

- What `lyx orch` is: the hub orchestrator hosted as an interactive shuttle run in the prime's reed session, cycled before its context fills.
- The verbs and where they run (prime only).
- The start prompt order.
- The watcher's lifetime and single-instance lock.
- The idle rules.
- The four-phase cycle with its restart rules and the never-clear-without-a-handoff guarantee.
- The `.lyx/orch/` layout.
- The migration from an operator's own terminal session: `/scribe:handoff` there, exit it, then `lyx orch start --handoff <that file>`.

Its residuals section states:

- The idle probe fails closed on anything it cannot read as an empty box, including Claude's greyed prompt suggestion.
- A `SendMessage` landing between the handoff turn's end and `/clear` is lost from context.
- The transcript and Stop-payload shapes are Claude Code internals, so usage degrades to unknown rather than failing.
- A threshold above the auto-compaction point lets Claude Code compact first.
- reed's resume path replays the launch's `--resume <session-id>`, which names the pre-clear session once a cycle has run, so a reed server rebirth resumes the older context.
  `lyx orch stop` plus `lyx orch start` relaunches from the last completed handoff instead.

An `## Open risks` section lists the two smoke-verified questions — background tasks surviving `/clear`, and a stable `SendMessage` address across it — as unverified, for card 14 to fill in.

`docs/overview.md`: add an **orch** entry to `## Modules` (`internal/orchcli` + `internal/orchengine`, `lyx orch start|status|cycle|stop`, one or two sentences, pointing at the package documentation) and the two packages to the directory tree listing.

`CONSTRAINTS.md`:

- Add `orchengine` to the Told-Geometry Invariant's bound packages.
- Add `orchengine` to the Config Strictness Invariant's degrading set.
- Add `lyx orch start` to the CLI/Cobra Invariant's interactive-handoff exception list.

No new invariant is introduced.

`manifest/roadmap.md`: add a short `## Done` entry for the hub orchestrator strand with automatic context cycling, pointing at the `internal/orchengine` package documentation.
