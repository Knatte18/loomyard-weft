# Gate tiers take a configured go test timeout via {test_timeout}

`internal/fabricengine` tier1 panics with `test timed out after 10m` when the machine is loaded (load average about 30, several tier1 runs at once).
The package takes about 349 s alone, and the gate passes no `-timeout`, so `go test` applies its own 10m default.
The cause of the package's slowness is out of scope; running it under `/dev/shm` was measured at only 2% to 7% faster, so the fix is a configurable timeout.

## What changes

- A tier command may hold a new placeholder, `{test_timeout}`.
  `renderTierCommand` in `internal/gatecli` fills it with the tier's `test_timeout_min` followed by `m`, such as `20m`, the duration form `go test -timeout` reads.
- A tier sets `test_timeout_min` in the hub's gate config beside its other keys.
  It defaults to `DefaultTestTimeoutMin` (20) when the command holds `{test_timeout}` and the key is unset.
- `LoadConfig` in `internal/gateslot` refuses `test_timeout_min` below 1, and refuses it on a command with no `{test_timeout}`, naming the key and `lym config gate`.
  This follows how `filter` is validated against `{filter}`.
- `internal/gateslot/template.yaml` and the `gateslot` and `gatecli` package docs describe the placeholder and the key.

## Design decisions

- This is a placeholder and a config key rather than a literal in the gate, so each hub sets its own bound and a tier whose command holds no placeholder behaves as before.
- `test_timeout_min` is not hashed by `tierFingerprint`, like `timeout_min` and `parallel`: a timeout change does not invalidate carried green runs, and no tier's existing fingerprint moves.
- The key is separate from `timeout_min`, which bounds the whole tier run once its slots are held; `test_timeout_min` is the test runner's own per-binary bound.
- No code path enables the timeout by itself: a tier takes it only when its command holds `{test_timeout}`.

## To check by hand

- This diff does not edit the hub's gate config, so the hub's tier1 command still lacks `-timeout {test_timeout}`.
  The hub's tier1 keeps its `GOFLAGS` `-timeout=20m` stopgap for now: a `lym` binary without this change would pass the placeholder through literally.
  After the landing's deploy installs the new binary, edit `tiers.tier1.command` to carry `-timeout {test_timeout}` and drop the `GOFLAGS` flag, then run `lym gate test --tier 1 ./internal/fabricengine` under load.
  This was left open by review: finding `tier1-command-lacks-test-timeout` (MEDIUM).
- Whether 20 minutes is the right default against the roughly 600 s seen under load.
