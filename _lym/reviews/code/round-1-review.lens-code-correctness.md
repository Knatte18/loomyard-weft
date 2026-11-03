```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran the added and changed functions on concrete inputs. Cases marked "ok" held on the code I read.

**Per function or branch, with the case run**
- `endingOf`: ok on all five cases. A wait error with a stop gives `stopped`. A wait error alone gives `wait error: …`. A done run with a failed gate gives `gate failed`. A done run with a passed gate gives `done`. A non-done outcome with a stop gives `stopped`. A died or timed-out run gives its outcome name.
- `seatRun.watch`, assistant branch: ok. The marker is written once, after `Wait`. A write error is logged and dropped.
- `stopAdvisors` with `markStopping`: ok for a live assistant, which reads `stopped`. An assistant already done keeps `done`. A seat with a nil handle (not started) is skipped, and its marker came from `notStarted`.
- `Run` with `SkipReason` set: ok. It writes a `not started` marker and starts nothing.
- `Run` with a start error on an assistant: ok. It writes a `not started` marker and goes on.
- `Run` with `clearAssistantFiles`: ok. It removes the marker and outputs before the chair starts, and a missing file is no error.
- `Resume` with a found assistant: ok. It attaches and writes the marker at its end.
- `Resume` with a lost assistant: ok. `markLostAssistant` keeps an existing marker and writes `not found` when none exists.
- `Validate`, kinds, directives and markers: ok. Advisors or writers mixed with assistants are refused. A missing directive is refused. An empty marker, a marker shared between seats, and a marker equal to an output (all seats' outputs are in `Table.Outputs()`) are refused. The missing-engine-block pass-over applies only to `seat.Assistant`.
- `CheckMarkers`: ok. It passes over a missing block for an assistant alone.
- `seatValues`, assistant: ok. The parent name is empty, which gives the no-parent variant. The edit directive comes from `EngineDirective` for the seat's own engine.
- `assistPaths`: ok for a review under `_lym`, a review elsewhere, and a review directly in `_lym`. A retry token `1b` gives different file names.
- `withAssistant` and `assistRulesBlock`: ok. The `"\n\n"` lead before the blank line matches the stencil's `{{.cluster_rules}}{{.assist_rules}}`. A round with no assistant renders an empty optional marker, so the bytes are unchanged.
- `keepAssistReports`: ok. It copies only when the marker equals `done`. An unreadable marker is logged and copies nothing.
- `readRoundAttempt`: ok. A missing record, an undecodable record, or an empty `Attempt` reads as the bare round with no assistant.
- `recordRoundAttempt` and `pick`: ok. `pick` clamps the round to the list length, and an empty model gives an empty fan.
- `ValidateAssistModelList`: ok. It accepts an empty list and an empty entry, and refuses a bad entry naming module, key and entry.
- `resolveAssistModels`: ok. An empty entry gives the zero choice.
- `CapAssistTimeout`: wrong for one input, see F2.
- `Review` with a failed `composeAssist`: wrong, see F1.

**Findings**

- **F1 — MEDIUM, design.** `internal/burlerengine/assist.go:100-130` (`composeAssist` and `assistSeat`), with `internal/seatengine/table.go:241` and `internal/burlerengine/prompt.go:71-76`.
  - When composition fails, `composeAssist` returns the zero `reviewFilePaths`. The assistant seat is still built, and `reviewSeatValues` then puts blank `instruction_1_path` and `instruction_2_path` into its `Values`.
  - `Table.Validate` refuses a blank seat value (`validateValues` at table.go:241), so `Run` fails for the whole table before the chair starts.
  - The documented behaviour is that a failed composition (a missing engine block or an unknown fan) makes the assistant not start while the chair goes on. Instead the round errors out.
  - The `SkipReason` path never gets to write a marker, because `Validate` runs first.
  - Fix: when `skipReason` is set, give the seat non-blank placeholder instruction paths in `Values`, or have `assistSeat` fill them from the skip. Alternatively, make `Validate` ignore blank values for a seat with `SkipReason`.
  - Add a test with an unknown assistant fan that expects the chair to run and the marker to read `not started`.

- **F2 — LOW, design.** `internal/loomengine/review.go` (`CapAssistTimeout`, the `want` branch) with `internal/seatengine/table.go` (`Seat.Timeout`: zero takes the table's timeout).
  - `assist_reviewer_timeout_min: 0` is accepted, since only values below 0 are refused. `CapAssistTimeout` returns a zero bound with no skip reason when `room > 0`.
  - A zero `Seat.Timeout` means the table's timeout, which is the chair's. The assistant then runs uncapped and can outlast the chair's wait bound.
  - The same zero bound happens when `room > 0` and `want == 0`.
  - Fix: treat `want == 0` as the capped `room`, or refuse it at load.

- **F3 — LOW, design.** `internal/loomengine/review.go` (`CapAssistTimeout`, the chair fallback to `runTimeout`).
  - If `review_timeout_min` is 0 and shuttle's `run_timeout_min` is also 0, `room` is at most 0. The assistant gets a skip reason and never runs, with a Warn at every wiring.
  - I did not read what shuttle means by a 0 `run_timeout_min`. If 0 means unbounded, the cap should not apply.
  - Fix: confirm the meaning of 0, and if it means unbounded, skip the cap in that case.

**Tool calls**
1. `grep` on `table.go` and `prompt.go`: I needed `Table.Outputs`, where seat values are validated, and `reviewSeatValues`, none of which the diff shows.
