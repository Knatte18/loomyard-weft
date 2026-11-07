```yaml
lens: code-tests
outcome: reported
```

**Per-behaviour list (lens: code-tests)**

- **`ResolveWithAnchor` and `resolveWithAnchorCore` removed** (`internal/lymcwd/lymcwd.go`): this is a deletion, so no new behaviour needs a row.
  - Nothing is left to test, and my first search in the review (`grep -rn ResolveWithAnchor`) found no remaining reference.
  - `ResolveWorktree` still reaches `resolveCore` unchanged.
  - `internal/lymcwd/lymcwd_test.go:128-148` pins that `ResolveWorktree` returns the recorded subpath with no gate error.
  - `lymcwd_test.go:178-184` pins its stale-marker refusal.
  - `lymcwd_test.go:220-225` pins symlink handling.
  - `gate_test.go` still pins the `buildLocation(applyGate=false)` path. The diff dropped no coverage of a surviving function.
- **Comment-only edits** (all of `cmd/lym`, `internal/adoptcli`, `render.go` and the docs): no behaviour changed, so no row is needed.
  - The `render.go` header claim "a blank `gateTiers` renders none" describes existing code, `gateTiersOrNone` at `render.go:607-613`. The diff did not change that code.
  - I found no test asserting that behaviour in `internal/websterengine/*_test.go`. It is pre-existing and not added by the diff, so I raise no finding.
- **Test economy:** the diff adds no test and no assertion on prose.

**Findings:** none.

**Tool calls (1 of 6):**
1. `grep` for `ResolveWorktree`, `gateTiersOrNone` and `"none"` in the `internal/lymcwd` and `internal/websterengine` tests. My context did not show which tests pin `ResolveWorktree` or the `gateTiers` fallback.
