# ly-drive: re-step transient and post-deploy stops, park at hand-back

A loom-launched ly-drive driver now clears the two stops a single re-step would clear, without the hub, and no longer exits at a hand-back.
Every other stop is still handed back, and `awaiting` and `paused` are never auto-resumed.

## What changes

- **Transient mark.**
  `internal/shedengine` declares a typed transient mark, and shed persists it with the `failed` state.
  The step error envelope carries it as a `transient` key holding the class name, empty when not transient.
  The refusal-kind vocabulary stays at five; `transient` is a key, not a kind.
- **Closed class list:** `git-transport`, `github-api` (transport failure or 5xx) and `agent-start` (`shuttleengine.ErrNotStarted`, now surfaced on shuttle's Result).
  Rejected pushes, auth failures, 4xx responses (including rate limits) and gate verdicts are never transient.
  The new `internal/shedtransient` package translates the classifiers in `gitexec` and `githubclient` into the mark at the producer and bootstrap boundary (`NewShed` and both step bootstraps).
- **Build identity.**
  Every `lyx shed step` records `vcs.revision` and `vcs.modified` in its step record.
  `lyx shed status` reports them on `last_step` and derives `binary_changed`: true only when both identities are known and differ.
- **Park marker.**
  `internal/shedrun` declares a marker file in the run's scratch directory.
  A parked driver sets it; `lyx loom start` consumes it.
- **`lyx loom start` resumes a parked driver.**
  A live driver with the marker gets one `Send` of a resume line that names a fresh drive-report path, and nothing is spawned.
  A live driver without the marker gets neither a send nor a spawn.
  A dead driver has any stale marker removed and is respawned as before.
  If the pane is not ready within a bounded wait, the command refuses, names a retry, and leaves the marker.
- **Launch prompt and `ly-drive` skill.**
  A loom-launched driver parks at a hand-back and runs the teardown command only at `done` and `busy`.
  The skill gains the closed re-step list, the parked state, the binary watch, resume handling and repair records under `<scratch_dir>/repairs/`.
- **Docs:** `SKILL.md`, `lyx loom start` help, `docs/overview.md`, and a new Transient Stop Invariant in `CONSTRAINTS.md`.

## Design decisions

- The driver never matches error text.
  Classification happens in Go at the failure site, and anything unrecognised is not transient, which is today's behaviour.
- A transient stop gets one immediate re-step, separate from the two-repair cap on trace repairs.
  A repeat stop at the same producer is handed back.
- A binary-change re-step is eligible at `blocked`, and at `producer`, `bootstrap` and `unseeded` errors the driver escalated.
  It is skipped when the warp has uncommitted changes (`git status --porcelain`, never reading the weft), and is not retried when the tree later turns clean.
  Because the re-step records the new identity, it happens once per deployed binary with no counter.
- The binary watch is a background shell loop on the `lyx` executable's mtime, confirmed by one `lyx shed status` read.
  Polling `lyx` would write a trace per poll, and a long-running `lyx` process would block the deploy's overwrite on Windows.
- The Go driver, orchestrator forks and operator-launched sessions do not park.
  Forks and operator sessions get only the transient and stop-time binary re-steps.
- The marker filename is declared once in Go, and a test pins the skill's copy to it.

## Check by hand

- Batten end to end: a parked driver on an awaiting child is resumed by batten's `Spawn` after `lyx loom approve`, and a `done` child's driver still exits before teardown.
- A real deploy while a driver is parked: one re-step, then the marker and watch are cleared.
- An unrelated deploy re-steps every parked, eligible run once, which costs one agent run per LLM producer.
