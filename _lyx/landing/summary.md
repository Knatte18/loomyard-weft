# Move the Go driver and tackler into shed; batten drives through it

The Go driver loop and the tackler leave `internal/loomcli` for a new shed-layer package, `internal/sheddriver`, so loom, darn and batten share one driver and one tackler.
Any later recipe gets them by supplying seams.
No default changes: `driver_kind` stays `llm` for loom and darn, so no existing run starts going through the Go driver.

## What changes

- `internal/sheddriver` holds the driver loop, rule table, settle records, stop reports, hand-over, pause hold and the tackler.
  It is a told-geometry engine: it imports no `lyxcwd`, `loomengine`, `loomshed` or `<module>cli`, and the arming module fills a seam struct for the recipe-specific parts (interrupt policy, arbiter, hand-back, verify goto target, watched read, friction note, hand-over spawn, tamper snapshot, resume goto targets, recipe description).
  An unwired optional seam removes a row or ends in a hand-back or a skipped check.
  The rule table's rows and order, the repair cap and the verdict vocabulary are unchanged.
- `lyx loom drive` keeps its name and mounts the `sheddriver` body with loom's seams; it serves darn runs too.
  The arbiter stays loom's code and is reached through a seam.
- `lyx batten run` now drives its own run through the `sheddriver` body in-process, so an unexpected batten stop meets the rule table and a tackler before the orch.
  Batten arms the until-stop loop without the agent idle watchdog and without an arbiter, takes the driver lock for the process's life, and adopts a still-running detached step on a rerun.
  Its tackler runs in the prime's reed session and may resume only the halted row.
  A hand-back queues one orch notice per stop episode, sharing the rows' per-episode marker; a requested pause sends none.
- A new `shed` config module (`shed.yaml`) holds `driver_kind`, `tackler`, `tackler_timeout_min` and `tackler_max_spawns`.
  `loom.yaml` keeps its four keys, `darn.yaml` gains the four, and `batten.yaml` gains the three tackler keys; each is an override that inherits `shed.yaml` when empty or absent.
  `tackler_max_spawns: 0` in a module turns that module's tackler off; a `tackler_timeout_min` of 0 inherits, and 0 in `shed.yaml` takes 10 minutes.
- Each run's `driver_kind` is seeded from its own recipe's module override, then `shed.yaml`, at both seeding sites; a typed `--child-driver` still wins and a recorded driver never changes.
- The tackler stencil moves to `contracts/stencils/shed/shed-template-tackler.md`.
  It is recipe-neutral, names no arbiter, requires a `{{.recipe}}` marker, lists the stops it receives with their usual verdicts, and gives a reading order (envelope and stop report first, traces only when needed).
- The settle records move from loom's friction directory to a per-run `shedrun` path; the driver, `arbiterstop`'s hold reading and batten's publish retry follow.
  No run holds records at the old location, so there is no migration.
- Docs follow: `docs/overview.md`, package docs, the refusal spec, and PATTERN (`PATTERN-shed-driver` added; `PATTERN-told-geometry`, `PATTERN-shed-verb-set` and `PATTERN-driver-choice-single-site` updated).

## Design decisions to know

- `loom.yaml` in existing hubs spells every old default out, which would make `shed.yaml` dead for loom.
  `loomengine`'s migration therefore rewrites the four keys to the inherit value once, and only when all four hold the old template defaults together with the old `driver_kind` comment.
  A separate step refreshes any of the four comments that still equal the old template text, leaving values untouched.
  Both steps are idempotent, and `run_settings` documents are never touched.
- Batten wires no tamper snapshot, because the prime has other writers and a snapshot would read `tampered` falsely.
  The stencil's write rule (verdict and case brief only) is prompt text that nothing enforces.
- A batten tackler `resume` names only the halted row, so it does what a rerun of `lyx batten run` does today, Land and Worktree-Teardown included.
- A new-build `lyx batten run` started while an old-build batten process holds the run lock exits as busy; a rerun after that process exits resumes at the recorded row.
- `lyx batten run`'s `Long` text was trimmed under the 2000-byte cap, dropping the sentence on a second batten being refused by the run's own lock.

## To check by hand

- The tackler has never run live; stencil and rule-table faults will surface on the first live go-driven run.
- A batten tackler that writes files, stages changes or commits in the prime leaves that state in place, and a later Land can carry it into the push and deploy.
- A killed batten step child is now an `interrupted` stop: Run-Shed re-steps under the repair cap, and a third kill at the same stop goes to the tackler.
- Batten's hand-back notice and the rows' notices share episode bookkeeping; confirm the orch gets a stop once.
- Migration on a hub where one of the four keys was pinned off the old defaults: the key keeps its value and the new comments apply.
- Go-driven loom and darn runs have no message receiver, as before; `orch-split` owns that.
- Not run by hand: a live `lyx loom drive` over a go-seeded loom or darn run, and a live `lyx batten run` over a blocked or error stop.
