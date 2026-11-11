```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens on the F1–F8 change set. I made no tool calls.

**[LOW] Stale test doc comment.**
- Changed text: the doc comment of `TestGateTestMapLinkedPackages` in `internal/gatecli/testmap_integration_test.go`.
- It still reads "The hub's packages caller and dirtier link linked."
- The fix added a `follower` fixture package and a `follower=linked` entry in `testMapLinksEnv`, but the comment was left as it was.
- Subtests now depend on `follower` linking `linked`: the cap test and the fresh-read test.
- It also names an enumerated list of consumers, which the prose rules discourage; "the hub's packages link `linked`" would stay true.

**[LOW] Ambiguous "such a file" in the `mapPackages` doc.**
- Changed text: the added clause in the `mapPackages` doc comment in `internal/gatecli/testmap.go`: "and one whose linked package holds such a file just before its build".
- The preceding sentence says "uncommitted or untracked Go file", but the linked-package check counts any owned file. The fixture's `linked/notes.txt` is not a Go file.
- "such a file" therefore reads as "Go file" and understates the rule.
- The Long help text and `doc.go` say "modified or untracked file" correctly.

Everything else agrees:
- **`--max` rules:** the Long help text, the `doc.go` line 1004 addition, the `mapPackages` and `mapPackage` docs and the `mapPackages` code all say that only the before-build linked skip counts nothing.
- **After-tests and changed-file skips:** these still count, matching `doc.go` line 1023.
- **Sibling test:** `SkippedPackageCountsAgainstMax` is a moved-file skip, so it still holds.
- **`mapPackage` callers:** `mapPackages` is the only one.
- **`doc.go` line 987:** "twice afresh per package it builds" remains accurate.

Further tool calls: none.
