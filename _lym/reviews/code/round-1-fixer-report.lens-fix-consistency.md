```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the `internal/shuttleengine` row change in `_lym/landing/summary.md` (6 → 5) agrees with the rest of the Removed table.

I compared every row's net-lines figure with the `git diff --numstat 9c5bd131..HEAD` output already in context, as deleted minus added:

| Package | Computed net | Table |
|---|---|---|
| `internal/runfsm` | 9 + 12 = 21 | 21 |
| `cmd/lym` | 1 | 1 |
| `internal/burlerengine` | 48 | 48 |
| `internal/shedadapters` | 33 + 8 = 41 | 41 |
| `internal/fabricengine` | 11 + 12 = 23 | 23 |
| `internal/loomshed` | 12 | 12 |
| `internal/websterengine` | 11 | 11 |
| `internal/orchcli` | 10 | 10 |
| `internal/shuttleengine` | 8 − 3 = 5 | 5 (changed) |
| `internal/reedengine` | 8 | 8 |
| `internal/battenshed` | 5 | 5 |
| `internal/reportengine` | 5 | 5 |

Every figure now matches. The "What" cell of the `shuttleengine` row (`skillStartFixture`, and `skillStartFixtureWith` whose doc comment "now stands alone") agrees with the diff and the commit log. The prose elsewhere in the summary cites no line counts, so no twin of the 6 is left stale.

No findings.

Further tool calls: none. The context already held the numstat output, the review and the summary.
