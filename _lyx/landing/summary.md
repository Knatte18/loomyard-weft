# Run records tolerate newer binaries; a hung verify is not re-run

A run record written by a newer `lyx` binary no longer halts or loses data in a long-lived batten or loom process started on an older one.
A verify step that hung is no longer re-run unfixed, and `lyx gate test` stops a CPU-busy Go test binary long before go test's 10-minute timeout.

The incident behind the first half: batten halted at Run-Shed with `decode failed: json: unknown field "routed_to"`, because a newer loom run had added a field to `status.json`.
The incident behind the second: a livelocked test spun at about 600% CPU, the orch stopped it with SIGQUIT, and the gate started a second identical run with no fix in between.

## Run records

- `internal/state` reads with unknown fields accepted.
  `ReadJSONStrict` loses `DisallowUnknownFields` and is renamed `ReadJSONNoMkdir`, for what still sets it apart from `ReadJSON`: it creates no missing parent directory.
  The rename touches many call sites across `battencli`, `loomcli`, `loomshed`, `shedengine`, `shedverbs` and others.
- `WriteJSON` and `UpdateJSON` read the file they replace under the lock they already hold, and carry over each key the record's Go type does not declare (`internal/state/keep.go`).
  The merge recurses through structs, pointers, maps and slices.
  Types with their own `json.Marshaler`/`Unmarshaler`, `json.RawMessage`, interface values and non-struct values are written whole, so `board.json` encodes exactly as on `main`.
- An array element keeps its unknown keys only while its known fields are unchanged.
  An appended or untouched history entry keeps its keys; an entry updated in place, removed or moved loses them, so a key is never carried onto a different element.
- `shedrun`'s seed decode and `battencli`'s cleanup status judge also stop refusing unknown fields.
  `seedWire` and its `LegacyParent` field are removed; an agreeing `WriteSeed` leaves an existing seed untouched.
- `PATTERN-tolerant-records` pins the rule, and a `go test` scan fails on any production `DisallowUnknownFields` call.
  The scan covers `encoding/json` only; the `yaml.v3` decoders of agent-written contract files stay strict on purpose.

## Hung verify steps

- The webster verify gate skips its flake rerun when the first run hung: a timeout, an exit code of 128 or above, an empty-detail exit of -1, or a log with go test's timeout panic, the runtime's `SIGQUIT: quit` header, a test binary ended by a signal, or the watchdog line.
  The hung result goes to the fixer and is never recorded as flaky.
  A rerun that itself hangs is reported the same way.
- The gate report, the fixer's findings and the stuck reason say the step hung and was not re-run.
  The loom and landing verify gates name a watchdog quit, its window and `busy_quit_sec` in their halt reasons.
  The matching `contracts/specs/refusal-spec.md` rows gain that way forward.

## CPU watchdog in `lyx gate test`

- On Linux, while go test runs, `lyx gate test` samples the CPU time of every `.test` process in its child tree every 5 seconds (`internal/proc`, loop in `internal/gatecli`).
  A binary at 90 percent or more of one core for a continuous `busy_quit_sec` window gets one SIGQUIT and a line on stderr naming it, so go test fails the package with goroutine stacks.
- `busy_quit_sec` is a new hub-wide `gate.yaml` key: absent reads as 120 seconds, 0 turns the watchdog off, a negative value is refused.
- No watchdog runs with `-bench` or `-fuzz`, or under an enclosing `lyx gate test` whose process is a live ancestor, so no binary gets two SIGQUITs.
  Other platforms get a no-op that logs once.
- Package docs of `state`, `gatecli`, `websterengine`, `loomshed`, `landingshed` and `proc`, and the gate template, describe the above.

## Design points for review

- Kept keys pass through only from the file being replaced, so no caller can add a key this way.
  A kept value can go stale when an older binary changes the known fields it depends on, such as `paused_since`.
- A healthy but CPU-bound package that runs over the window is quit and read as a hang, which the fixer cannot fix in code; the operator raises `busy_quit_sec` through `lyx config gate` from the prime.
- The exit-code and text markers can over-match (a test printing a marker, git's fatal 128); all that costs is the one rerun.
- The guarantee starts with this binary: processes started on an older binary still refuse or drop new fields until restarted.
- A livelock below 90 percent of one core, or a deadlock without CPU, still runs to go test's `-timeout`.

## To check by hand

- The verify gate failed once on `TestLoadState_UnusualFiles/legacy_integration_records_still_load` in `internal/websterengine/state_test.go`: it expected a save to drop the retired `assertedModel` key, which every state write now keeps.
  Commit `067b74ec9` flipped the assertion; confirm that is the intended behaviour.
- Check that a finalize-time verify failure has its own refusal-spec row (`finalize-verify-failed-no-refusal-row`).
- Check that `loomshed`'s round-record rewrite keeps unknown keys (`loomshed-round-record-rewrite-drops-unknown-keys`).
- Check that alignment's `attempt.json` is written through `internal/state` (`alignment-attempt-json-bypasses-state`); a later commit routes it there.
- The rename commit included a scripted heredoc append to `internal/state/state_test.go`; an edit audit may flag it.
