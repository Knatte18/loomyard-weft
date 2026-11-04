```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: I found two LOW findings and nothing at MEDIUM or BLOCKING.

**LOW: `internal/battenshed/primelockwait.go:1-2` (F6 header).**
- The new header says "Worktree-Teardown calls it once, and Worktree-Create's HoldPrimeLock calls it on each hold."
- `land.go:244` also calls `acquirePrimeLock`, so the header still understates the callers.
- The old header had the same gap. The finding named only Worktree-Create, so the fix carried it over.
- A reader who relies on the header would miss the Land row's wait.
- Suggested wording that avoids listing callers: "the bounded, cancellable wait that every producer holding the prime lock calls in place of a single Acquire."

**LOW: `internal/fabricengine/add_hold_test.go:1` (F1 file header).**
- The file header still says only "the caller's lock is taken once and released by the time Add returns."
- The test now also pins that the hold is taken before the worktree is made and released after it.
- The header is not wrong, but it no longer describes what the file proves.

**Checked and consistent, no finding:**
- **F4:** the `create.go` `HoldPrimeLock` comment and the `doc.go` Worktree-Create paragraph now match each other, and neither pins line numbers.
- **F4, `wire.go` comment:** the comment at `wire.go:939-940` does not claim "never around a network call", so the review's "if it states the same" branch did not apply. Leaving it was correct.
- **F4, `Add` comment:** the `add.go` comment near line 204 says "the hold spans none of it", which refers to the probes' round of network. That matches the new wording.
- **F5:** the new sentence on `AddOptions.Hold` agrees with the `Add` doc comment, which says Add takes the teardown lock first.
- **F2:**
  - The takes and releases counts match the code paths. A fresh create takes 1, because `refuseHeldMember` returns before its hold for a slug outside a beam. A held member takes 2 after `Add` and 1 over a complete pair.
  - The remaining `holdNothing` callers are unaffected: `wire_seams_test.go`, `beamland_integration_test.go`, and the other rows of `beamfork_integration_test.go` and `lifecycle_integration_test.go`.
- **F2, `TestRefuseHeldMember`:** the updated doc comment matches the new held-beam row.

Tool calls made:
1. `grep -rn "acquirePrimeLock("` over `internal/`, excluding tests. I needed this to check the new `primelockwait.go` header's claim about who calls it, which my context did not show.
