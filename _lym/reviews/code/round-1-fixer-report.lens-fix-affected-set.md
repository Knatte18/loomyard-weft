```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the F1 fix covers every row of the Removed table in `_lym/landing/summary.md`, not only the `internal/shuttleengine` row the review named.

Findings: no findings.

I checked each row against the `git diff --numstat 9c5bd131..HEAD` output already in context. The net figure is deleted lines minus added lines, summed per package.

| Package | Numstat | Net | Table |
|---|---|---|---|
| `internal/runfsm` | state.go 1/10, state_test.go 0/12 | 21 | 21 |
| `cmd/lym` | 0/1 | 1 | 1 |
| `internal/burlerengine` | 1/49 | 48 | 48 |
| `internal/shedadapters` | 0/33, 0/8 | 41 | 41 |
| `internal/fabricengine` | 0/11, 0/12 | 23 | 23 |
| `internal/loomshed` | 0/12 | 12 | 12 |
| `internal/websterengine` | 0/11 | 11 | 11 |
| `internal/orchcli` | 0/10 | 10 | 10 |
| `internal/shuttleengine` | 3/8 | 5 | 5 (fixed from 6) |
| `internal/reedengine` | 0/8 | 8 | 8 |
| `internal/battenshed` | 0/5 | 5 | 5 |
| `internal/reportengine` | 0/5 | 5 | 5 |

Every changed file in the numstat belongs to one of these twelve packages, so the table leaves out no package. No other cell in the summary restates the shuttleengine figure, so the 6 to 5 edit has no stale twin.

Further tool calls: none. The numstat output and the edited summary were already in context.
