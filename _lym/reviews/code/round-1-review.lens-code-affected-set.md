```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change the diff makes, I list what it forces outside the lines it touches.

**Per change, the set found**

1. **`Add` now deletes a git-ignored-only directory at the task worktree path instead of refusing with `*ErrTaskPathTaken`.**
   - `ErrTaskPathTaken`'s type comment (`internal/fabricengine/addadoptonly.go:73`) says "the task worktree's directory already exists". Its `Error()` text at lines 80–87 is also unqualified. Neither says the refusal now applies only to a directory with something real in it. Not covered, LOW (F1).
   - The two consumers of `ErrTaskPathTaken` are `internal/battencli/wire.go:393` and `internal/battencli/readoptstop.go:31,140`. Their wording "save anything worth keeping, then delete it" is still true for non-ignored content. Covered.
   - The existing refusal test `add_adoptonly_integration_test.go:377` writes a plain file to the path. That file is neither a directory nor ignored, so `removeIgnoredTaskPath` returns false and the refusal still holds. Covered.
   - Pre-flight ordering: **not covered, see F2.**
   - `doc.go:1000` and the `Add` doc comment at `add.go:175–177` are both updated. Covered.

2. **A new `ErrStrayPathKept`-wrapped error now leaves `Add`.**
   - Consumers: `remove.go:285` and `pairteardown/teardown.go:479` use `errors.Is` only on `Remove`'s own error. `pairteardown` is not an `Add` caller. No `Add` caller tests for the sentinel. Nothing is broken.
   - The `ErrStrayPathKept` doc (`remove.go:52`) and the `doc.go:788–792` list name only `Remove` and `RemoveStrayTaskPath` as sources. `Add` is not in either. The `Add` doc comment (`add.go:175–177`) does not mention the new failure error. Not covered, LOW (F3).

3. **A new third exception in the destruction chokepoint.**
   - `pattern/PATTERN-fabric-boundary.md` and `doc.go:1318–1331` both carry "three exceptions". Covered.
   - Both sit in the pattern file's Destruction chokepoint section. The ownership order in the executor rule reads "containment, ownership, dirtiness, force". `removeIgnoredTaskPath` reaches `removeWorktreeLeftover`, where `os.Root` containment still applies, so the bullet is consistent.
   - The `destroy.go` header (lines 1–6) still describes only the gate executors. It is not wrong.

4. **New raw git call `gitexec.RunStdin(check-ignore)` in `destroy.go`.**
   - `PATTERN-gitrepo-client-boundary` requires `gitexec.Run` or `runChecked`, raw only at `//gitexec:raw` sites. `RunStdin` is the same `gitexec` entry point and is already used in `fabricengine` (`snapshotuncommitted.go:237,269`, `integrateweft.go:245`, `taskpatch.go:105`). No violation.

5. **`Pattern-Departure:` trailer.** The commit messages carry none and none is needed, since the PATTERN change is declared in the diff itself. No declared-PATTERN-change list was supplied beyond the `pattern/` diff, and it lands.

**Findings**

- **F2, MEDIUM, `design`.** `internal/fabricengine/add.go:338–348`. The delete runs before the later pre-flight refusals, so those refusals no longer happen "before any mutation".
  - **What runs after the delete and can still refuse:** `list warp remotes` and `no remote configured` (350–356), `ErrRecordsWorktreeNotPair` (366–384), `refuseAdoptLocalCheckedOut` (388–396), the parent and HEAD checks (402–414), `warpDefaultBranch` (419), the weft probe error (434–436), `ErrNotLive` (443–461) and the anchor refusal (465–468).
  - **What contradicts the delete:** the `Add` doc comment says `ErrNotLive` and the adopt refusals are "refused before any mutation" (`add.go:181`). The comment at line 470 marks the point where "the pre-flight refusals have passed". The deferred `regenerateAfterVerb` is registered only at line 471. Neither sees the delete.
  - **Result:** the delete happens, the path is gone, and `Add` then refuses with a different error. `KindPathRemoved` sits in `res.Mutations`, and the refusal paths return `AddResult{}`. The deferred snapshot at line 209 still copies the recorder, so the entry is not lost.
  - **Fix:** move the `os.Stat`/`removeIgnoredTaskPath` block to just after line 471, or make the `Add` doc state the exception to "before any mutation".

- **F1, LOW, `consistency`.** `internal/fabricengine/addadoptonly.go:73` and the `Error()` text at 80–87 still describe `ErrTaskPathTaken` as "the task worktree's directory already exists". Qualify them to a directory `Add` will not delete (not a registered worktree, a `.git` entry, a non-ignored file, or not a real directory).

- **F3, LOW, `consistency`.** `add.go:175–177`, `remove.go:52` and `doc.go:788–794`. `Add` can now return an error wrapping `ErrStrayPathKept`, and none of these says so. Add one line to the `Add` doc comment and the `doc.go` bullet.

**Tool calls**
1. A grep of non-Go files for `ErrTaskPathTaken`, plus reads of `addadoptonly.go:70–95`, `battencli/wire.go:385–410` and `add_adoptonly_integration_test.go:355–385`, because my context lacked the error's definition, its consumers and the existing refusal test.
2. A read of `PATTERN-gitrepo-client-boundary.md` and a grep for `RunStdin` in `fabricengine`, because the git-boundary rule and the precedent for the new call were not in my context.
3. A grep for `ErrStrayPathKept` consumers and a read of `doc.go:780–796`, because the sentinel's callers and its doc listing were not in my context.
