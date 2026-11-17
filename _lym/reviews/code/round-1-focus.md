---
round: 1
focus:
  - "internal/lymtemp/sweep.go `makeTreeOwnerWritable` and `removeTreeMakingWritable`: does the walk stay safe and correct when a directory is unreadable, a link points outside the tree, or an entry vanishes mid-walk, and does a chmod or ReadDir failure abort the removal where os.RemoveAll alone would have succeeded?"
  - "internal/lymtemp/root.go: the `remove` field now defaults to `removeTreeMakingWritable`; does every other caller of that field, including the retry on directory-not-empty in `removeRunDirectory`, still behave as before, and does the sweep with apply false still remove nothing?"
  - "Goal asks for the sweep of the cleanup verb to make a leftover tree owner-writable: does the diff cover the scratch sweep that `internal/cleanupcli` runs, or only the `lymtemp` dead-run-directory sweep, and does the `/tmp/lym-1000` leftover case from the Goal actually reach the changed code?"
  - "Goal asks that tests needing a read-only directory take it from one testkit helper: do the remaining `os.Chmod` sites in test files (for example `internal/websterengine/rebaseline_test.go`, `internal/fabricengine/hook_test.go`, `internal/creelengine/server_test.go`, `internal/pairteardown/archive_test.go`, `internal/dotnetimpact/derive_test.go`, `internal/lymtemp/root_test.go`) make a path read-only or unreadable and restore it only in `t.Cleanup`, or are they in-test restores or files that the Goal does not cover?"
  - "Converted test sites (every `_test.go` under the diff that now calls `permkit.Chmod`): is each conversion behavior-preserving, including tests that restored the mode mid-test (the helper's cleanup restores the original bits, not a fixed 0o755), tests that remove the path before the end, and ones that chmod with a mode other than the read-only case?"
  - "internal/testkit/permkit/permkit.go: `Chmod` records `info.Mode().Perm()` only; does the restore lose setuid, setgid or sticky bits, and does the helper work on a file as well as a directory? Check the Windows behavior against the package's own claims."
  - "internal/testkit/permkit tests and `testmain_test.go`, and the new `lockedTree` row in internal/lymtemp/sweep_test.go: do they cover the new behavior without duplicating each other, and does the row skip or tolerate Windows and root runs where permission bits do not bar removal?"
  - "Test economy: `internal/logger.TestSweep_DeleteFailureTolerance` is listed as contained, covered by `TestSweep_AgeBound`; compare their assertions before filing anything, and apply the rubric's keep-or-fold rule."
  - "PATTERN.md, pattern/PATTERN-test-isolation.md and internal/lymtemp/doc.go: do the added lines match what the code does, and does the PATTERN.md index line stay consistent with the background file's text?"
---

Seed rationale: the answer key is the board entry alone, with two ideas: a sweep that makes a leftover tree owner-writable, and one testkit helper for read-only paths.
The focus entries ask where the diff may fall short of each idea and where the conversion of many test sites could change behavior.
