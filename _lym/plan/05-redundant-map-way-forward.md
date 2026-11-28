# Card 5 — redundant-map-way-forward

**Edit:**
- `internal/gatecli#redundantCmd`
- `internal/gatecli#runRedundant`
- `internal/gatecli#TestGateRedundant`

**Uses:**
- `internal/gatecli/doc.go`

**Intent:** Name `lym gate test --map` as the way forward of every `lym gate redundant` refusal that names a map rebuild (D9).
`runRedundant`'s `rebuild` text becomes `way forward: build it with lym gate test --map <package> from this worktree, then re-run the same command`, and serves the refusals for a map that is missing, unreadable or records only a failed build.
The refusal for a map built from other Go files than the worktree holds says first to commit or revert the worktree's edits to the package, since a map run measures no package with a modified file, then to build it with `lym gate test --map <package>` from this worktree.
The inputs reaching them are `TestGateRedundant`'s fixture maps: `nomap` (no map), `badindex` (unreadable), `broken` (a failed build only), `stale` and `lacking` (other Go files).
`redundantCmd`'s `Long` text and `runRedundant`'s comment name `lym gate test --map` where they name `lym gate testmap`; `internal/gatecli/doc.go` already names the flag from card 1.

Tests: `TestGateRedundant`'s refusal rows gain a `wantWayForward` field: `lym gate test --map nomap`, `--map badindex`, `--map broken`, and for `stale` and `lacking` both `commit or revert` and `lym gate test --map <package>`; `OutsideEveryHub` and `AFailedKeepScan` keep their own ways forward.

**ImpactSummary:** Refusal text of one verb in gatecli and its test rows; no caller parses the text.
