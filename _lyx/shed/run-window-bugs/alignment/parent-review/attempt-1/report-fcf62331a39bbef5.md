---
verdict: APPROVED
---

# Alignment report — run-window-bugs at parent-review

## Scope against the board entry

The decision record judged is the current one on disk, the rewrite with D1 to D7, whose Goal reads "Fix the seven defects bundled on board entry `run-window-bugs` so none recurs".
An earlier version with D1 to D8 was replaced while this review ran.

Every bundled item has a decision, and none does work belonging to another entry:

- `shuttle-ready-blank-pane`: D1 reads readiness from `SessionStart` plus a turn start or tool event ahead of the capture, which is the entry's first bullet.
  The entry's second bullet offers a minimum pane height or scrollback capture as means, so that "a collapsed strand cannot read as never-ready".
  D1 meets that end without either means, since after D2 every `start` launch carries a first turn, and the entry's closing line ("readiness must not depend on a pane capture alone") holds for `start`.
  Wait's and attach's probe keeps the capture rule, with its reason recorded; the sightings were spawns through `start`.
- `shuttle-color-verified-send`: D2 does the one remaining bullet, which is that the launch line carries the skill-load message.
- `loom-checklist-stale-render`: D3 does a full clear on every redraw and wakes on resize, and an acceptance criterion pins the shorter, narrower frame drawn over a longer, wider one, as the entry asks.
- `weft-push-refused-retry`: D4 answers the three "What to find" questions (ref `main`, a remote race, retry is the intended path) and records the finding.
  The finding that the three Publish sightings were `PushWithRetry`'s own retries is an auto-mode assumption, not read from each run's Publish trace as the entry's first question asks; Discussion-Review can weigh it.
- `reed-socket-test-fd-sweep`: D5 does the sweep, and puts the helper in a test kit.
- `agent-gofmt-verb`: D6 adds a `lyx` formatting verb and an edit directive line, which is the first of the entry's two options.
- "Teardown races a reed lock": D7 makes teardown wait for the late writers with a bounded retry, a form of the entry's "waits for" option.

No direction the entry rules out is taken.
No other board entry duplicates or contradicts these decisions.

## Overlap with runs in flight

`lyx loom peers` lists no landed parent commits and two peers, `judge-cap-call` (Merge-In) and `go-driver-assistant` (Webster).

- Go package `internal/loomcli`: D6 adds `lyx loom fmt` there, as a new verb file, a registration in `cli.go` and a section in `doc.go`, which the record's same-commit docs list names.
  Both peers change this package, `internal/loomcli/doc.go` included.
  Neither peer lists `cli.go` or a `fmt` file.
  `go-driver-assistant`'s record already has `judge-cap-call` taking its `loomcli/doc.go` changes at Merge-In.
- Go package `contracts/stencils`: D2 changes a comment in `contracts/stencils/roleskills_test.go`, and D6 changes `contracts/stencils/edit/edit-directive.md`, which `stencils.go` embeds.
  Both peers change `contracts/stencils/stencils.go` and `contracts/stencils/claims_test.go` in this package; neither lists `roleskills_test.go` or `edit/edit-directive.md`.
- `go-driver-assistant`'s tackler renders the edit directive, so it will pick up D6's new line; nothing in its record conflicts with that line.

The only file all three change is `internal/loomcli/doc.go`; the rest of the overlap is at package level.
No decision of either peer contradicts or duplicates one here.
Neither peer changes `PATTERN.md`, so D2's rewording of `PATTERN-role-skills-typed` touches no peer's PATTERN entry.

The record's last plan-writer note says `go-driver-assistant` "edits the same `shuttleengine` files".
That peer's changed paths list no `internal/shuttleengine` file, so either the note is wrong or the peer has not reached those files yet.

## Shared bookkeeping, named and not escalated

- `docs/overview.md`: D2 changes its skills line, and `judge-cap-call` also changes the file.
- `cmd/lyx/testdata/test-budget.yaml`: both peers change it, and this run will if its new tests pass a package's row.
- `contracts/specs/refusal-spec.md`: both peers change it, and D6's refusals may add rows to it.

## Why this escalates

The design has no scope deviation.
The escalation rule names a Go package that this run and another run both change, and `internal/loomcli` and `contracts/stencils` are each such a package, shared with both peers.
The parent decides whether this run may go on beside them, and in what landing order.
