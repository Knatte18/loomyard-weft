```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set (F1–F6 fixes) for consistency with its surroundings.

**Findings**

1. **LOW** — `internal/driftengine/capture.go:75` (the `RunCapture` doc) and `internal/driftengine/doc.go:126`.
   - Summary: both docs still describe the old backup behavior after `backUpBaseline` changed.
   - Evidence: both say "the first free numbered backup beside it". `backUpBaseline` now takes the first number where neither `baseline.N` nor `baseline.N.meta` exists, and removes the metadata directory beside an empty baseline.
   - The `backUpBaseline` doc comment was updated, but its two siblings were not, so they are slightly stale.
   - The `doc.go:126` package doc also doesn't mention that stale metadata beside an empty baseline is cleared.

No other inconsistencies found.
- The `unchangeableWayForward` change (dropping `then lym loom resume`) has no stale twin. `doc.go:368` names only `lym drift amend <slug> --entries` with `metadata` marks, which still agrees.
- The `NewDriftGate` doc and `doc.go` claim that findings name records and seal faults. The Terminal path now does so through `faultLines`.
- `nextFree` has no remaining references.
- `Findings()` output is unchanged, since it uses the same extracted `faultLines`.

Tool calls (2 of 6):
1. grep for the way-forward text and for `nextFree` and backup wording across `.go`, `.md` and `.yaml` files. My context lacked the docs and other call sites outside the files read.
2. grep of `doc.go` and `contracts` for backup and `.meta` wording. My context lacked `doc.go`.
