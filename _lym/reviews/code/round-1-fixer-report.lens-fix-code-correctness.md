```yaml
lens: fix-code-correctness
outcome: reported
```

**Findings**

- **LOW** — `fronttask.File`, the logged-and-swallowed record write: after a failed record write, `Remove` of that front task drops no record and runs no `on_land` action until a `Settle` has recorded it. The window is narrow and the review fix text accepts it.
- **LOW** — `TestHubSettleDeps_OwnedHereReadsThePrimeOwnerOnTheHubBoard` (F5): only the prime branch is exercised; the `beam != ""` branch (`owners.Effective` with the `hubgeom.ParentOf` closure) is untested.

Traced and sound: F3 (empty report gives the fixed reason; the new row fails with the defect), F4 (reorder correct; new subtest fails with the old order; `ParentOf` now runs for every non-done task entry), F2 (the `brokenRecord` row fails with the defect), F1 (`startDetached` keeps the `testing.Testing()` guard; the test pins arguments, directory, output file, spawn log, pid and the unknown-key refusal).

Tool calls: none.
