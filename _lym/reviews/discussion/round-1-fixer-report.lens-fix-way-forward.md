```yaml
lens: fix-way-forward
outcome: reported
```

Task: check, through the fix-way-forward lens, that every refusal, error, next step, row or test this fix added can be reached from the state it names and leads to progress.

**MEDIUM: an error report blocks its own retry on an unchanged tree.** This concerns two pieces of new D7 text:
- "It is skipped when the worktree's existing report matches the current files by D8's match, so a repeat call on an unchanged tree runs no build and no test."
- "every error it meets is a line in the report, never an exit code"

Together they leave a report holding a transient error (a failed coverage build, a failed `lym` build, a corrupt map, an exhausted budget) or a "no map" line valid for as long as the tree stays the same. Every later gate call skips the step, so the step never runs again and D8 keeps showing that report to Code-Review. Nothing says how to get a fresh run: the tree has to change, or someone deletes the report by hand.
- The fix states only one exception: "A report written on a red tier records no hashes and never matches." Error-bearing reports and no-map reports get no such rule.
- D8's new map-rebuild bound covers contained verdicts going stale. It does not cover a no-map report after a refresh builds the map, because that report has no verdict for review to read.
- Suggested change: only a report whose packages were all judged, with no error line and no "no map" line, counts as matching. Otherwise the step runs again.

**LOW: a skipped package's report gives no way out.** This concerns the new D5 text: "an untracked Go file therefore makes a package skipped, never stale", together with D6's new bound that such a package "waits for the next prime HEAD change".
- An untracked Go file in the prime survives every HEAD change, so the package is skipped on every refresh and never gets a map.
- The new exit output reports "packages it skipped" only as a count. It names no package or file and no next step, such as committing or removing the file.
- Suggested change: the skip report names each skipped package with its cause (the dirty file, a held lock, or HEAD moving mid-package).

Every other way forward the fix added can be reached and leads to progress:
- **No parent branch:** the D7 step 1 report line for this case is reachable.
- **Restarted dobby:** it reads the live `test-map` record and starts no second refresh. `runproc`'s lister checks `isAlive`, so a dead child's record does not block.
- **Discard under a landing:** the landing itself moves HEAD, so D6 retriggers.
- **Failed report write:** with no report, the next call runs again.
- **Reinterpretation:** it runs once per hub behind a marker and mails the orch, through the existing hubreconcile path.
- **New acceptance criteria:** each one is reachable with a stand-in `go`.

Further tool calls: none.
