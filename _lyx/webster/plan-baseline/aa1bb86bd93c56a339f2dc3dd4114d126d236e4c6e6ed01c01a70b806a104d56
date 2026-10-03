# Card 5 — tier-purity-hermetic-env

**Edit:**
- `cmd/lyx/tierpurity_test.go`
- `cmd/lyx/tiersleep_test.go`
- `cmd/lyx/hermeticenv_test.go`
- `cmd/lyx/gitkitspawn_test.go`

**Uses:**
- `internal/testkit/scankit/scankit.go`

**Intent:** Convert the Test Tier Purity and Hermetic Git Test Environment scans to `scankit` per D2 and D4, following the overview's `scan-conversion-protocol`.
Both drop the `go env GOMOD` exec and the `exec.LookPath("go")` skip;
`tierPuritySkipDirs` is deleted in favour of `scankit`'s shared skip-dir set, which cards 3 and 4 already moved its other readers onto.
`allowedSpawners`, `allowedLongSleepers` and `allowedNonHermetic` become one-entry-per-line `{entry, reason}` lists (D8) with stale-entry detection.
The stale detection retires every `allowedSpawners` entry whose only reason was the `go env GOMOD` exec that cards 3 and 4 removed (D4);
any other stale entry it finds is removed too, and all are named in the commit body.
An entry kept because its file still carries banned tokens as scan data keeps a reason that says only that.
The banned-token sets, `knownTierTags`, the gitkit spawn definition and the hermetic presence token stay verbatim.
Prove each scan by one planted violation (an untagged file with `exec.Command`, a git-spawning package without `HermeticGitEnv`) and remove the plant.
`CONSTRAINTS.md` needs no edit: neither invariant's text names the retired entries.

**ImpactSummary:** Four `cmd/lyx` test files; the two scans' verdicts are unchanged apart from removing stale allowlist entries.
