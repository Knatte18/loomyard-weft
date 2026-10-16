# Go driver and tackler in shed, for every FSM-driven run

`twb-btf-ks` · Note · enhancement · running · Discussion-Bouncer

# Go driver and tackler in shed, for every FSM-driven run

`shed-tackler` · Task · enhancement, shed, loom, batten · running · Worktree-Create

## Seen (2026-10-14)

The Go driver and the tackler landed in `1c812c1f8` for loom runs only.
Their code is in `internal/loomcli` (`drive.go`, `tackler.go`), and the keys `driver_kind`, `tackler`, `tackler_timeout_min` and `tackler_max_spawns` are loom config.
Darn has no `driver_kind`, so a darn run always has an LLM driver.
Batten steps its own shed run in Go but spawns no tackler: an unexpected stop there goes straight to the orch, and batten only reads whether a loom child's tackler or arbiter is in flight to hold its notice.

## Wanted (operator, 2026-10-14)

Any FSM can go wrong, so every shed-driven process needs the tackler, not only loom: darn, batten, and any later recipe.
The tackler, and the Go driver it sits behind, belong in shed as code.
Shed's own config holds the default tackler model and budgets.
A module's own config can override them, the way loom's keys do today.

## Order (operator, 2026-10-14)

The Go driver and tackler are not tried on a live run until they are moved into shed; the first live test runs on the shed version.

## How the tackler is told what to do today

`internal/loomcli/tacklerprompt.go` fills the stencil `contracts/stencils/loom/loom-template-tackler.md` with paths and types it in as the whole launch prompt.
The tackler reads the shed stop envelope, the Go driver's stop report and log, the stop step's trace copy and stderr, the child traces, and any read-only `lyx` command or file in the worktree; it writes only the verdict file (`resume`, `hand-back`, `escalate`) and, on an escalate, the case brief for the arbiter.
The stencil is already recipe-blind: the recipe and row arrive only through the envelope.
Its loom-specific parts are its name, the driver log path and the arbiter hand-off.
A shed version can keep one stencil for every FSM, with an optional module-filled marker, such as a short description of the recipe, for context the envelope does not give.

## Stencil points to weigh (orch read, 2026-10-14; no agent review yet)

- It has never run on a live run (`driver_kind` is set nowhere); only Go tests exercise it.
- It gives no examples of the stops it receives (transient errors, `producer`, `bootstrap`, `unseeded`); a short list of how each typically looks and its usual verdict would help a small model.
- It sets no reading bound: the child traces can be large for Haiku; reading the envelope and stop report first and traces only when needed would help.
- Being recipe-blind, it lacks context on what the recipe does; see the optional module marker above.

## Related

`orch-split` (its "Was `go-driven-shed`" section) wants the same LLM involvement at every level; this note is the shed part of that.
