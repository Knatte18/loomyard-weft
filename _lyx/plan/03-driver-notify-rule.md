# Card 3 — driver-notify-rule

**Create:**
- `contracts/stencils/shed/shed-template-driver-notify.md`
- `contracts/stencils/shed/shed-template-driver-notify-watched.md`

**Edit:**
- `internal/shedrun/paths.go`
- `internal/shedrun/paths_test.go`
- `contracts/stencils/shed/shed-template-driver.md`
- `contracts/stencils/stencils.go`
- `contracts/stencils/claims_test.go`
- `internal/loomcli/driverprompt.go`
- `internal/loomcli/driverprompt_test.go`
- `internal/loomcli/start.go`

**Uses:**
- `internal/proc/proc_linux.go`

**Intent:**
D6, render side: the driver stencil renders its parent-notification rule by whether a live batten watches the run, decided in Go at the driver's fresh launch.

`internal/shedrun/paths.go` declares the watched marker beside the park marker: an exported `BattenWatchedFileName` (`batten-watched`) and `BattenWatchedMarker(l, runID)`, `BattenWatchedFileName` under `ScratchDir(l, runID)`, so it sits under `.lyx` (`PATTERN-lyxdirs-single-declarer`, `PATTERN-durable-vs-ephemeral-state`).
The file holds the watching batten's pid in decimal on one line; card 8 writes it.
A row in `paths_test.go` pins the path beside `ParkMarker`.

The `## Notifying the parent` section body of `shed-template-driver.md` moves into two registered stencils, rendered into a new required `{{.parent_notify}}` marker under that heading, and the stencil's header comment names the marker:
- `shed-template-driver-notify`: today's rule, verbatim (the generic escalation line with the message tail, the verbatim `parent_notice` relay, the no-parent and failed-send fallback, the `done` with `friction: failed` exception);
- `shed-template-driver-notify-watched`: the batten-watched rule. The driver sends the parent a message only for an `awaiting` envelope's non-empty `parent_notice` (relayed verbatim with the tail, as today), a stop report that asks the parent a question the driver cannot settle itself (naming the run-id, the report path and the question, with the tail), and a `done` stop whose envelope reports `friction: failed`. It never messages the parent that the run stopped, halted or finished and sends no generic escalation line, an `awaiting` stop with no `parent_notice` included, because batten announces those stops. The failed-send fallback stays.
The other sections' cross-references to `## Notifying the parent` stay valid under both renders; reword any sentence there that would contradict the watched rule (for example "Escalations also notify the parent session") so it defers to that section.
Register both in `contracts/stencils/stencils.go` (embedded vars and registry rows) following the `parent-directive-*` variant precedent.
Move the driver's `## Notifying the parent` claims in `claims_test.go` onto `shed-template-driver-notify`, and add claims for the watched variant: it keeps `parent_notice`, `friction: failed` and the question message, and carries no generic escalation line.

`driverPrompt` gains a `watched bool` parameter and fills `parent_notify` with the watched or the plain stencil, read through `stencilstore.Read`.
A helper in `driverprompt.go`, given the marker path and an injected `func(pid int) bool` liveness check, reports watched only when the marker exists, parses as a pid and that pid is live; an absent, unreadable or unparseable marker, or a dead pid, is unwatched.
`startLLMDriverArm` in `start.go` computes it from `shedrun.BattenWatchedMarker(c.location, resolvedRunID)` with `proc.IsAlive` and passes it to `driverPrompt`.
It reads no seed (`PATTERN-driver-choice-single-site`), and the resume path still renders nothing, so the rule is fixed at the fresh launch.

Tests in `driverprompt_test.go`: rendered watched, the prompt has no instruction to message the parent on a stop, halt or done and no generic escalation line, and keeps the `parent_notice` relay, the question message and the `friction: failed` message; rendered unwatched it keeps today's rule; the helper reads a live-pid marker as watched and an absent marker, a garbage marker and a dead-pid marker as unwatched.

**ImpactSummary:** `driverPrompt` gains a parameter with one production caller (`start.go`) and its test; the driver stencil gains one marker and loses one section body to two new registered stencils; `shedrun` gains one path constructor.
