# Gate: TRX-based verdict, declared-key darn prompt, env override notice

`lym gate test` can now take a vstest tier's verdict from a TRX results file instead of console text.
It reports a crashed test host's cause, names a refused `--gate` key's alternatives, and flags tier env values that differ from the session's.
The darn writer prompt now names a measuring call the hub's gate config can run.

Closes #163
Closes #156
Closes #162
Closes #157

## What changes

- **Verdict from a TRX file (#163).**
  A tier with `counter: vstest` and `{args}` in its command may set `results: trx` (`internal/gateslot`, `internal/gatecli`).
  The gate puts `--results-directory <dir> --logger trx;LogFilePrefix=gate --blame` at the front of `{args}`.
  It reads counts and failed tests from the file, so a console logger among the call's own words cannot change the verdict.
  On a failed run, each run-level message in the results files, such as a test host crash, goes to stderr and the gate log and counts among the causes.
  A call whose own words hold a results directory or a `trx` logger is refused, since the gate owns them.
  A tier without `results` behaves as before.
  `results` is part of the tier fingerprint, so a change to it never carries a green pass over.
- **Unknown gate key (#162).**
  `Config.Gate` now names the keys the config holds when it refuses a missing one.
- **Darn prompt (#156, #162).**
  The gated Step 4 fragment's measuring call is a `{{.measure_call}}` marker.
  `Config.MeasureCall` renders it from the hub's gate config: `lym gate test --gate darn-write`, with a targets word only when a run tier takes `{targets}`.
  It refuses a key of scope card and a key that runs a tier whose command holds no `{args}`.
  `loomcli` loads the config and fills `DarnInputs.MeasureCall` only for a writer whose verify gate has a cold fixer; `DarnTable` requires it then.
- **Env override notice (#157).**
  Before a tier runs, the gate prints one line naming each tier env variable whose value differs from the session's, and heads the tier's log with it.
  It names the tier's value only, since the session's value may be a secret, and skips a variable the session leaves unset.
- `docs/dotnet-hub.md` and the `gate.yaml` template document `results: trx` and the `darn-write` key's `{args}` requirement.

## Design decisions

- `MeasureCall` has no `--tier` fallback.
  When Gate refuses the key it returns that refusal and the darn prompt render fails with it.
  A hub that cannot run `darn-write` fails its own verify anyway, and Gate's refusal names the config fix.
- `results: trx` is opt-in per tier, so existing `gate.yaml` files are unchanged.
- No red-first test covers the stencil change: the overlay cannot restore a non-Go file on the merge base.
  The Go tests reach the behavior through `DarnTable` and `loomcli` wiring.

## To check by hand

- Nothing was run against a real .NET hub.
  Run a `results: trx` tier with a detailed console logger among the `--` args, and one whose test host crashes, and read the verdict and gate log.
- Review findings left open:
  - `darn-measure-call-halts-ungated-runs`: confirm `wireDarn` in `internal/loomcli/darnwiring.go` never loads the gate config or fails for a run without a cold fixer.
  - `duplicate-tier0-trx-tests`: check `internal/gatecli` (`trx_test.go`, `tierrun_spawn_test.go`, `test_integration_test.go`) for tier0 TRX tests that repeat one another.
  - `measurecall-renders-refused-call`: confirm `Config.MeasureCall` in `internal/gateslot/tiers.go` never returns a call the gate would refuse.

## Merge danger

**Door:** two-way
**Blast radius:** gate
