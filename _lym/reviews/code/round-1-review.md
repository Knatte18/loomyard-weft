---
verdict: APPROVED
---

The diff makes the change the board entry's second bullet asks for and leaves the first bullet's behavior as it already stood on the base.

- **Dirty paths on every dirty end.** `fixerEndDirty` is produced at one place, `fixerSession.judge` in `fixerrun.go`, and `outcome.dirty` is set right before the halt there. `run` copies `session.outcome`, so the paths reach `haltReason`. `parentalign.go:421` builds only a spent outcome, so no caller can build a dirty outcome with an empty list.
- **Cap.** The dirty reason goes through `listShown`, the helper `dirtyReason` and the pull-request review reasons use, so the cap matches the package's other dirty-tree reasons.
- **First bullet.** Commit `b5e0ed26` is an ancestor of the base `1fbac34f`. On this branch `judge` re-prompts a dirty tree on a charged turn while `Spent < cap` and halts only at `Spent >= cap`. The incident logged `spent=1 cap=2`, so it predates that commit, as the summary says. The landing gate refuses a dirty tree, so keeping the post-last-turn dirty end a halt does not contradict the entry. `doc.go` already states the re-prompt at the fixer-turn sentence.
- **Tests.** The dirty row asserts the path (`after its last turn: a.go;`), the session log, the way forward and the absence of the old wording. `avoidReason` is an existing table field. The `reflect.DeepEqual` change is the only comparison the new slice field breaks.
- **`doc.go`.** The halt sentence names the dirty halt, says its reason names the paths, and still says every halt opens with `LandingStopPrefix`. It applies to Parent-Align and Finalize alike, since both pass the outcome whole to `haltReason`.
- **Summary claims.** The timeout and no-report ends keep their wording in the split-out case, and the dirty row asserts the old generic wording is absent. Both bear out against the diff.
- **Repository rules.** The diff touches no `PATTERN.md` entry and declares no departure. The new reason names a way forward and a test reaches it, as `PATTERN-refusal-way-forward` asks.

## Rejected

- The code-affected-set lens's LOW note that `s.halt(fixerEndDirty, dirtyReason(point, dirty))` passes a findings text that differs from the new halt reason: the lens itself calls it no defect, the findings text goes to the seat run and not to the operator, and the old code did the same.
- The code-tests lens's observation that no row has more dirty paths than `dirtyPathsShown`: the cap is `listShown`'s own behavior, already used by the package's other dirty-tree reasons, so a row here would be a per-helper test the rubric's test economy rule discourages.
