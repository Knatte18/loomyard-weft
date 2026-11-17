---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/lymtemp/sweep_test.go:91
    summary: The sweep's claim that a link inside a dead run directory is never followed by the owner-writable pass has no test row.
    fix: In internal/lymtemp/sweep_test.go TestSweep, extend the lockedTree case (or add a sibling kind) so the dead run directory also holds a link, made with fslink.CreateDirLink, to a directory outside the root that permkit.Chmod sets to 0o555. After the apply sweep, assert that the outside directory still exists and its permission bits are still 0o555.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/lymtemp/sweep.go:397
    summary: A chmod or ReadDir failure in the owner-writable pre-pass returns before os.RemoveAll runs, so a tree RemoveAll alone could remove is now left in place and reported as a failure.
    fix: Make makeTreeOwnerWritable best-effort in removeTreeMakingWritable. Call it, discard its error, and return os.RemoveAll(path), so the removal's own error is what the sweep reports. Inside makeTreeOwnerWritable, keep walking the remaining entries when one subdirectory's chmod or ReadDir fails rather than returning at the first error. Update the two doc comments and the doc.go paragraph to say the pass is best-effort. Add a TestSweep row whose dead run directory holds a subdirectory the pass cannot list, with the chmod pass failing while RemoveAll still succeeds, or failing through the default remove, and assert that the outcome is the one RemoveAll gives.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: pattern/PATTERN-test-isolation.md:182
    summary: The new rule that permkit imports only the standard library sits in the list that enforcement_test.go is said to enforce, but no test enforces it, unlike its scankit sibling.
    fix: In internal/testkit/enforcement_test.go, add TestEnforcement_PermkitStdlibOnly, modelled on TestEnforcement_ScankitStdlibOnly, calling scankit.AssertImportAllowlist(t, "internal/testkit/permkit") with the same tiergate tier and with //lym:guard and //lym:watch internal/testkit/permkit/*.go directives.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/logger/retention_test.go:308
    summary: The redundancy report lists TestSweep_DeleteFailureTolerance as contained by TestSweep_AgeBound, but it pins the nil return and the kept stale file on a failed unlink, which TestSweep_AgeBound never asserts.
    fix: Directly above the func TestSweep_DeleteFailureTolerance line in internal/logger/retention_test.go, add the line //testtiming:keep pins that Sweep returns nil and keeps the stale file when its unlink fails.
---

### [MEDIUM] The owner-writable pass's link safety has no test

**Location:** `internal/lymtemp/sweep_test.go:91` (the `lockedTree` case), `internal/lymtemp/sweep.go:425-428`

**Issue:** `makeTreeOwnerWritable` skips every entry whose `DirEntry` is not a directory, and `lymtemp/doc.go` and the function's doc state that a link is never followed.
If that ever regressed, the sweep would chmod directories outside the temp root.
That property is the safety bound of a pass that changes permissions.
The only link row, "prefixed link", covers a top-level link, which `ownerDead` rejects before `remove` runs.
No row puts a link inside a dead run directory.

### [LOW] A pre-pass failure aborts a removal that would have succeeded

**Location:** `internal/lymtemp/sweep.go:397-400`, `:416-424`

**Issue:** `removeTreeMakingWritable` returns any non-not-exist error from `makeTreeOwnerWritable` without trying `os.RemoveAll`.
Two cases hit this: a subdirectory whose chmod fails with EPERM, such as one a run under sudo owns, and a 0o700 directory of another owner whose `ReadDir` fails.
Before the diff, `os.RemoveAll` removed such a tree whenever its contents were removable, for example an empty foreign-owned directory under a writable parent.
The case is rare under a 0700 per-user root, but the pass was added to widen what the sweep removes and should never narrow it.
No row drives a pre-pass failure through the default `remove`.
Every error row replaces `root.remove`, so the fix adds the row that pins the chosen behavior.

### [LOW] permkit's stdlib-only rule is listed as test-enforced but unenforced

**Location:** `pattern/PATTERN-test-isolation.md:182`, `internal/testkit/enforcement_test.go:81-88`

**Issue:** The background says `internal/testkit/enforcement_test.go` enforces the import rules listed above it, and the diff adds permkit's stdlib-only rule to that list.
`TestEnforcement_ScankitStdlibOnly` pins scankit's matching rule, but nothing pins permkit's.
The rule matters, because `lymtemp`'s tests import permkit, and `tmuxkit` imports `lymtemp`.

### [LOW] TestSweep_DeleteFailureTolerance needs a keep mark

**Location:** `internal/logger/retention_test.go`, `TestSweep_DeleteFailureTolerance`

**Issue:** The redundancy report lists the test as contained by `TestSweep_AgeBound`.
`TestSweep_AgeBound` asserts only that the old file is absent and the recent one exists.
This test asserts that `Sweep` returns nil when an unlink fails and that the stale file stays, a failure path the covering test never reaches.
Under the rubric's keep-or-fold rule it is kept, and it needs the mark.

## Focus verdicts

- `makeTreeOwnerWritable` safety: unreadable directories are opened, because the bits are set before `ReadDir`. A link is not followed, because `DirEntry` types come from the link itself. Vanished entries are filtered at each hop. The one defect is F2.
- The `remove` default: `removeRunDirectory` is its only caller. A pre-pass error is not the not-empty symptom, so it is reported after one attempt, and the retry is unchanged. `Sweep(false)` never calls `remove`.
- Scope against the Goal: not a defect. The Goal locates the leftovers under `/tmp/lym-1000`, the lymtemp root, and the cleanup verb's `temp` segment (`internal/cleanupcli/segments.go:141-158`) sweeps that root through `Root.Sweep`.
  `TestCleanupVerb_ScratchSegmentSweepsTheAnchorOfASubpathHub` runs `--apply` over every segment. Under `tmuxkit.Main` the process root resolves to the real `/tmp/lym-<uid>`, so the failing case reaches the changed code.
  The `.scratch` segment holds no test temp trees, so leaving it unchanged is consistent with the Goal's intent.
- Remaining raw `os.Chmod` sites in tests: the Goal does not cover them.
  Each is one of three kinds: a file set to mode 0 with no restore (`websterengine/rebaseline_test.go`, `dotnetimpact`, `pairteardown`, forkaudit's transcript, `creelengine`'s log), an in-test restore inside a remove seam (fabricengine), or a chmod that sets a mode rather than denying access (`vscode` write helper, `hook_test.go`, `lymtemp/root_test.go`, forkaudit's date stub).
  None makes a directory read-only and restores it only in `t.Cleanup`.
- Converted sites: every conversion preserves behavior. Restoring the original bits instead of 0o755 still leaves a `t.TempDir` tree owner-writable. Sites whose code under test removes the path rely on the dropped restore error, as permkit's doc states.
- permkit: it works on files and directories, the Windows skip reason in its test is accurate, and setgid and sticky bits are not restored (see Rejected).
- Tests: the `lockedTree` row pins the new removal and the dry run. On root the row passes without exercising the path, as every permission-based row in the repository does. On Windows the read-only attribute is cleared by the pass.
- PATTERN.md and its background agree with each other and with the code. `lymtemp/doc.go` matches `sweep.go`.

## Rejected

- code-correctness NIT, permkit restores only `Perm()` and drops setgid or sticky bits: the doc's "the bits path had before" refers back to the permission bits the same sentence names, the replaced code restored a fixed 0o755 that dropped them too, and removability is unaffected.
- code-tests LOW, no deterministic row for an entry vanishing mid-walk: staging the race needs a seam inside the walk that the production code has no other use for. The vanished-path filter on the walk's root is exercised whenever `removeRunDirectory` is called on a path a concurrent sweeper already removed.
- code-tests LOW, a chmod or ReadDir failure through the default `remove` is untested: folded into F2, whose fix adds that row for the corrected behavior.
