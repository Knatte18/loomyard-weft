# Run records tolerate newer binaries; hung verify steps are not re-run

## What changes and why

A long-lived batten or loom process can run on an older `lyx` binary than the one that wrote a run record, because prod moves at every landing.
A reader that refused an unknown field then halted the run: `plan-gate-fixes` stopped at Run-Shed with `decode failed: json: unknown field "routed_to"`.
Separately, a CPU-spinning test in a verify gate ran for minutes, and the gate then re-ran the same unfixed code and hung again.

### Run records

- `internal/state`: `ReadJSONStrict` no longer sets `DisallowUnknownFields` and is renamed `ReadJSONNoMkdir` for what still sets it apart from `ReadJSON` (it creates no missing parent directory).
  A malformed file is still `ErrDecode`.
- `WriteJSON` and `UpdateJSON` read the file they replace under the lock they already hold and carry over every key the record's Go type does not declare, recursing through structs, pointers, maps by key, and arrays by index.
  An array element keeps its unknown keys only while its known fields are unchanged, so a key is never carried onto a different element.
  Types with their own `json.Marshaler`, `json.RawMessage` and interface values are written whole, so `board.json`'s encoding is unchanged (pinned by a test).
- `internal/shedrun`: the seed decode is tolerant, and `seedWire` with its `LegacyParent` field is removed.
  An agreeing `WriteSeed` does not rewrite an existing seed, so its unknown keys survive.
- `internal/battencli`: the cleanup status judge accepts a status with an unknown key.
- `PATTERN-tolerant-records` plus a `go test` scan fail any production `DisallowUnknownFields` call.
  `yaml.v3` `KnownFields` on agent-written contract files is out of scope.

### Hung verify steps

- `internal/websterengine`: the verify gate skips its flake rerun when the first run hung and sends the failure to the fixer, never recording it as flaky.
  A hang is `TimedOut`, an exit code of 128 or above, a signal-killed shell, or a log marker (go test's `panic: test timed out after`, the runtime's `SIGQUIT: quit` header, go test reporting a signal-ended binary, or the watchdog line).
  The gate report, findings and stuck reason name the hang and say the step was not re-run.
- `lyx gate test` gains a CPU-busy watchdog (Linux only, via `internal/proc`).
  It samples every `.test` process in its child tree every 5 seconds and sends one SIGQUIT to a binary that used at least 90 percent of one core for `busy_quit_sec` (default 120, 0 disables, hub-wide `gate.yaml` only), so the goroutine stacks reach the log early.
  It is off for runs with `-bench` or `-fuzz`, and under an enclosing `lyx gate test`, found through a new environment variable that must name a live ancestor.
- `internal/loomshed` and `internal/landingshed` verify gates name the window and `busy_quit_sec` in their halt reason after a watchdog quit.
  The matching rows in `contracts/specs/refusal-spec.md` gain the way forward.

## Design decisions to know

- A healthy but CPU-bound package that runs past the window is quit and reads as a hang; there is no code fix, so the operator raises `busy_quit_sec` through `lyx config gate` from the prime.
  The halt reasons say so.
- The 90 percent threshold sits below one core because a single spinning goroutine yields samples just under one core at `/proc` tick granularity.
- Hang markers are matched as text anywhere in the log, and any exit code of 128 or above counts, so a false match only skips the one rerun.
- The guarantee starts with this binary; processes still running an older binary keep refusing or dropping new fields until restarted.
- A kept unknown key can go stale when an older binary changes the known fields beside it.
- A livelock below 90 percent of a core, or a deadlock without CPU, still runs to go test's `-timeout`.

## To check by hand

- Run `lyx gate test ./...` and `lyx gate test --tags integration ./...`; the tagged integration test drives a real spinning test binary with `busy_quit_sec: 1`.
- Nothing ran the watchdog against a real long hang in a hub, nor an older batten process against a newer `status.json`.
- The tests for undecodable `status.json` fixtures in `internal/loomengine` (and `internal/battencli`'s overview test) were vacuous once an unknown field decodes; they now use malformed JSON.
  Confirm each still fails to decode for the intended reason.
- The `internal/state` rename touched call sites in `battencli`, `battenrecipe`, `loomcli`, `loomrecipe`, `loomshed`, `pairteardown`, `shedcli`, `shedengine` and `shedverbs`; skim them for stray behavior changes.
- A verify gate failure fixed in `internal/websterengine/state_test.go`: `TestLoadState_UnusualFiles/legacy_integration_records_still_load` now expects a saved `state.json` to keep the retired `assertedModel` and `integrationFix` keys, because every state write now keeps unknown keys.
  Confirm that is the intended outcome.
- One test-file append in `internal/state/state_test.go` was made by a scripted edit; read it as ordinary test code.
- The `lyx-halt` `paused_since` key and a map value replaced under the same key keep stale or old unknown keys by design.
- `loomshed` round-record rewrites and the finalize verify-failed refusal row were flagged during planning; check that `internal/loomshed` rewrites of round records keep unknown keys and that the finalize verify-failed halt has a refusal-spec row.
