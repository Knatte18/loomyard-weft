```yaml
lens: fix-affected-set
outcome: reported
```

Task: the affected-set lens over F1 (return-false branches of `mapPackage`) and F2 (added/removed branches of `changedFiles`).

No findings.

- F1: `mapPackage` returns true only at `testmappackage.go:94`, the before-build skip. The after-tests dirty branch at `:134` and the changed-files branch at `:142` are the two other skip returns, and both return false. The review named both. `SkippedPackageCountsAgainstMax` already pinned the changed-files branch, and the new `ASkipAfterTheTestsCountsAgainstTheCap` pins `:134`.
- F2: `changedFiles` has two loops, one for removed or changed files and one for added files. The new `Added` and `Removed` rows cover both, through the one `changedName` path for a linked package. The own-package added and removed cases are not covered. The review's fix offered the linked variant as sufficient, so I count this as optional.
- The stand-in's `LandsLinked` case was the only caller of `appendAndCommit` that changed, and it now goes through `landInLinked`. The `Moves` and `Lands` callers are unchanged.

Unchecked: nothing.

Tool calls (1):
- Grepped the return sites of `mapPackage` and the callers of `changedFiles` and `dirtyOwnedFile`, because `testmappackage.go` was not in my context.
