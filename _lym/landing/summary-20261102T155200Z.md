# Gate tiers take a configured go test timeout through {test_timeout}

`internal/fabricengine` tier1 panics with `test timed out after 10m0s` when the machine load is about 30.
The gate passes no `-timeout`: the hub's tier1 command is `go build ./... && go vet {targets} && go test -json {targets} {args}`, so `go test` falls back to its own 10-minute default per test binary, and the package's wall under load passes that.

## What changes

- `gateslot.Tier` gains `TestTimeoutMin`, read from a new optional tier key `test_timeout_min`.
- A tier command may hold the new placeholder `{test_timeout}` (`gateslot.TestTimeoutPlaceholder`).
  `renderTierCommand` in `internal/gatecli` fills it with the minutes followed by `m`, such as `20m`, the form `go test -timeout` reads.
- A tier whose command holds `{test_timeout}` and sets no `test_timeout_min` gets `gateslot.DefaultTestTimeoutMin`, 20 minutes, which is well above the roughly 600 s package wall seen under load and below tier1's `timeout_min` of 30.
- `LoadConfig` refuses a `test_timeout_min` below 1, and one set on a command that holds no `{test_timeout}`, in the same way it refuses a `filter` without `{filter}`.
- `tierFingerprint` leaves the key out, like `timeout_min`, so a changed bound does not drop a green record.
- The gate template's comments and the `gateslot` and `gatecli` package docs describe the key and the placeholder.

## Design decisions

- The timeout is a placeholder that the hub writes into its own command, so no `-timeout` is hard-coded into the gate.
  The gate runs any shell command, `vstest` tiers included, so it cannot append a Go flag by itself.
- `test_timeout_min` is a key separate from `timeout_min`, because `timeout_min` bounds the tier's whole run, including the build, vet and every package.
  A per-binary bound below it lets `go test` panic with goroutine stacks before the gate kills the tree.
- The default is applied only when the command holds the placeholder, so a tier without it keeps `TestTimeoutMin` at 0 and its loaded config is unchanged.

## Hub config

As a stopgap until this change lands, the hub's tier1 `env` sets `GOFLAGS` to `-trimpath -p={parallel} -timeout=20m`, which an older binary also handles.
This change replaces that stopgap once the hub's tier1 command holds the placeholder.
Make the switch only after a lym binary with this change is installed, because an older binary would pass `{test_timeout}` to the shell as literal text.
The switch sets the command and drops `-timeout=20m` from the tier1 `GOFLAGS`:

```
lym config gate --set 'tiers.tier1.command=go build ./... && go vet {targets} && go test -json -timeout {test_timeout} {targets} {args}'
lym config gate --set 'tiers.tier1.env={GOFLAGS: "-trimpath -p={parallel}", LYM_TEST_TIER: "1"}'
```
