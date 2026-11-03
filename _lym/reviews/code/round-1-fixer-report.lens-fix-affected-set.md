```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Checked list:
- F3: every recipe-name list in the glossary now includes motif. A repo-wide grep of the docs found no other stale "tapestry, darn, sampler, pawl, batten" list. `docs/overview.md` already lists motif.
- F2, `commitStatusDeps` call sites: `wiring.go:319` and `wiring.go:957` use `c.commitStatusDeps`. The other `loomCommitStatusDeps` callers (`commitrecords.go:111`, `drivewiring.go:255` and `:260`) never read `FinishedStatus`. Only `newCommitStatusSeam` reads it, at `wiring.go:267`, so the missing finished status there is harmless.
- F2, the call sites themselves: the fix pins the `commitStatusDeps` method but not the two `c.commitStatusDeps(location)` call sites. A revert at either site still passes. This is the part of the review's F2 that the fix left open, and it is not a new finding.
- F1, F4: not set-premise findings, so I left them unchecked.

Calls made:
1. Grep of the docs for recipe-name lists, to find sites the glossary fix may have missed. My context had no repo-wide view of them.
2. Grep for `loomCommitStatusDeps(` and `c.commitStatusDeps(` call sites in non-test code. My context had only `wiring.go`.
3. Grep for `FinishedStatus` consumers and for `motif` in the glossary, to confirm the edits and that no other site reads the field.
