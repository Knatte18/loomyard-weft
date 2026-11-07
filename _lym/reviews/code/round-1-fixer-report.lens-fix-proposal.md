```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F1 proposal (take every template copy before the seam install) by building a case where it is wrong or breaks something.

No findings.

Cases I tried against the changed text (`boltOnlyOrigin`, `failingPullOrigin`, `parentPairOrigin`, `reconcilePair`, `InstallReplay` hoisted above `InstallWorktreeSeam`):
- **Hoisted copies share state.** Each subtest gets its own copy and mutates only its own, so an earlier subtest cannot leak into a later one. The seam events each subtest asserts are filtered by `within(event.Root, origin.Path)`, and a copy's path differs from the template's. So a build that no longer runs under the seam cannot add or remove an event those assertions count.
- **Moving `boardkit.InstallReplay` to the test's top.** It was installed once before, in the last subtest, and is installed once now. `TestReconcileOrigin_Moves` installs it at its top level over many subtests, so having it installed during the earlier clone and pull subtests is an existing pattern. The `movesTemplate` build now runs under the replay, as it did when the copy followed the old in-subtest install.
- **Cleanup order.** `t.Cleanup` runs last-in first-out, so the seam is uninstalled before the replay and before the copies' teardown. The copies' teardown therefore never runs under the fake seam.
- **A `-run` filter that skips subtests.** The hoisted copies are still built, which only costs time and changes no behaviour. The rubric excludes an improvement the fix did not make worse.
- **The remaining copy-then-install order.** `hubforge.NewHub(t, ".")` stays after the install, and the first subtest asserts the made events from that build, so it must run under the seam.

Tool calls: none; my context already held the review, the changed file and the helper definitions.
