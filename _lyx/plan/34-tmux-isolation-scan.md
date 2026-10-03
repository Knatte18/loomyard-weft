# Card 34 — tmux-isolation-scan

**Create:**
- `cmd/lyx/tmuxisolation_test.go`

**Edit:**
- `CONSTRAINTS.md`

**Uses:**
- `internal/testkit/scankit/scankit.go`
- `cmd/lyx/hermeticenv_test.go`

**Intent:** Enforce the Tmux Test Isolation Invariant (D18) with a scan on `scankit`, modelled on the Hermetic Git Test Environment scan.
For every package with an `integration`- or `smoke`-tagged `_test.go` file, it evaluates each test file's `//go:build` constraint (via `go/build/constraint`) under the untagged, `integration` and `smoke` tag sets, treating platform tags as satisfiable, and fails unless, under every set that compiles any of the package's test files, a file declaring `TestMain` compiles, and every `TestMain`-declaring file in the package calls `tmuxkit.Main`.
It carries an allowlist of `{entry, reason}` pairs, one per line, with stale-entry detection, and the allowlist starts empty;
an entry is admitted only for a package whose `TestMain` cannot call the entry point, its reason naming that obstacle, and "does not start tmux" is not a reason.
It has a floor of at least one tagged package.
Prove it by removing the `tmuxkit.Main` call from one package's `TestMain` (and, separately, deleting one `!integration` sibling), watching it fail, and restoring;
name the proofs in the commit body.
What this costs, bounded per D18: the rule keys on "has a tagged test file", not "starts tmux", so a tagged package that never starts tmux still carries the one call;
"starts tmux" has no static shape, and the call costs one temp directory.

`CONSTRAINTS.md` gains a Tmux Test Isolation Invariant section stating the rule, naming `cmd/lyx/tmuxisolation_test.go` as its enforcer, and stating that in an untagged run the entry point spawns nothing.

Before the PR, run `go test -tags integration ./...` and the smoke tier of any package whose smoke files changed, then confirm the default socket directory (`/tmp/tmux-<uid>/`) gained no socket from a test and no tmux server a test started is still running;
a new socket is excluded only when its key equals `reedengine.ServerName` of one of the operator's own hub paths.
Record the result in the commit body.

**ImpactSummary:** One new scan in `cmd/lyx` and one new `CONSTRAINTS.md` section; no production or existing test changes.
