```yaml
lens: fix-consistency
outcome: reported
```

Task: consistency check of this round's fix changes against their surroundings.

**LOW**: `_lym/landing/summary.md`, `## Merge danger`, the sentence "A `gates` entry that this binary rewrites without `reports` therefore turns the report steps back on". The claim is wider than the behaviour. An older binary fills `reports` only for keys whose own template lists them. In `git show main:internal/gateslot/template.yaml`, the grep for `reports` matched `batch`, `webster`, `parent-align`, `webster-fix`, `darn-write`, `code-burler-fix`, `harden-burler-fix` and `pr-review`. It matched no `card` line, and the other template keys it did not print have no list either. So rewriting one of those other entries turns nothing on. Qualifying the sentence as "a `gates` entry for which the older template lists reports" would fix it.

No other findings. These agree with the changed text:
- The new holder list in `internal/gateslot/doc.go` matches `pattern/PATTERN-test-speed.md:71`, `pattern/PATTERN-gate-slot.md:15` and the `HolderEnv` callers.
- The new `PATTERN-test-speed` bullet matches `cmd/testtiming/main.go`, where `exportTierSelection` runs before every mode, `-redundancy` included.
- The explicit `reports: []` rewrite that Merge danger names is possible: `gates` is a per-name map with no shape check (`ConfigSpec` in `internal/gateslot/config.go`).
- No other "re-run mark" clause about the removed gate behaviour is left.

Extra tool calls:
- `sed` over `internal/gateslot/config.go:120-200`: `ConfigSpec` was not in context.
- `git grep` for re-run mark, non-tier holder and card-verify runner sites: twins elsewhere were not in context.
- `grep` of `cmd/testtiming/main.go` and `redundancy.go` for the tier selector: the redundancy mode's env was not in context.
- `sed` over `cmd/testtiming/main.go:150-268`: whether the selector is set for every mode was not in context.
