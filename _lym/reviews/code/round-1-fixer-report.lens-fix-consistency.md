Findings:

1. **LOW**, `pattern/PATTERN-owner-machine.md:29` (Passes bullet, unchanged). It says "A resume that judged its owner without origin leaves the origin-owed marker, which stays until a pass that reached origin exits 0". The new exception also judges the owner without origin, but it writes no marker.
   - The fix added the exception to line 20 and to the line 27 bullet, but not here, so a reader of this bullet finds the rule unqualified.
   - The new line 21 does say "writes no origin-owed marker", so this is a stale sibling and not a contradiction.
   - The reviewer's fix key named only line 20 and the "Origin reconcile first" section, so the miss is minor.
   - Suggested change: append "bar a Worktree-Teardown resume whose task worktree is gone" to the bullet.

2. **LOW**, `pattern/PATTERN-owner-machine.md:3` (header sentence, unchanged). It says "the owner verdict that gates a resume, a wake or a landing judged against origin inside `ReconcileOrigin`", and the line 13 bullet repeats this. The new exception judges a resume on the local board with no origin.
   - The header already lists "a resume with origin unreachable reading the local Bolt" as a local-board case.
   - The new case is not listed, though line 21 now states it. This is a minor omission.

3. **No finding** on `PATTERN.md:27`. The index entry makes no claim about the reconcile, so it still holds.

4. **No finding** on `internal/battenshed/doc.go:62`. The added sentence agrees with `ownerPreflight` in `owner.go:240-243`.

5. **No finding** on `internal/battencli/owner_test.go`. The file header names the teardown pre-flight and the pass exit codes, matching `TestPassExitCode`.

Calls made:
- `grep -rn "PATTERN-owner-machine" --include=PATTERN*.md .`: located the `PATTERN.md` index entry, which my context lacked.
