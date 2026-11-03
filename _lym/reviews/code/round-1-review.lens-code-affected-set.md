```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set — find what the dirty-end halt-reason change forces outside the touched lines.

**Per change, the set found**

- `fixerOutcome.dirty` field. Struct literals: `fixerOutcome{}` and `fixerOutcome{end:, failureLog:}` at `parentalign.go:421` use named fields, so they are unaffected. The `!= fixerOutcome{}` comparison in `fixerrun_test.go:410` was the only comparison the slice breaks, and it is converted. Covered.
- Dirty-end `haltReason` wording. Callers: `finalize.go:804` and `parentalign.go:394`, both pass the outcome from `run()` whole, so `dirty` is carried. Only `fixerrun.go:262` sets `fixerEndDirty`, and it now sets `outcome.dirty` first. No other constructor of a dirty outcome exists. Covered.
- Old wording "ended without a usable turn (dirty)". A grep of non-test `.go`, `.md` and `.yaml` finds it only in the timeout/no-report case, which is intended. No stencil, doc or spec asserts the old dirty wording. Covered.
- `doc.go` halt sentence. It names the dirty halt, and `LandingStopPrefix` still opens every halt. Covered. One inconsistency: the new halt reason says "left uncommitted or untracked paths ... after its last turn" and uses its own way-forward wording rather than `dirtyReason`'s. That is not a defect.
- `PATTERN.md` and `pattern/`. The diff declares no PATTERN change. `PATTERN-refusal-way-forward` covers webster, shed and loom refusals. The new reason names a way forward (`lym loom resume`) and a test reaches it. Covered.
- Commit message. It has no `Pattern-Departure:` trailer, and none is needed.

**Findings**

- LOW, consistency, `internal/landingshed/fixerrun.go:262` and `:334`. `s.halt(fixerEndDirty, dirtyReason(point, dirty))` still builds a findings text from `dirtyReason`. It is the terminal gate result's findings, and the halt reason now comes from `haltReason`, so the two texts differ. The old halt reason also did not use this text, so nothing regressed. Raised only as a note: the `Findings` string is not user-visible in the halt. This is not a defect, and I recommend no change.

No other findings.

**Tool calls**
1. One grep over non-test `.go`, `.md` and `.yaml` files for `haltReason(`, "without a usable turn", `fixerEndDirty` and `end=dirty`. My context lacked the callers outside `fixerrun.go` and the doc/stencil references to the old wording.
