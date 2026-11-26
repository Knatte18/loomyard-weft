```yaml
lens: fix-way-forward
outcome: reported
```

Task: check the ways forward this fix introduced: the restore path, the installer calls and the new tests. Each one has to be reachable and has to make progress.

Ways forward the fix introduced:
1. The installer's restore, used to clear a seam for the unset-seam refusal test (D13, D5).
2. `internal/hubforge` calls the installer when it builds a hub (D13).
3. Tests outside fabric that read the workspace file install the IDE module's builder themselves (D13).
4. The code-tip rewind goes through shed's adapter's `state.WriteJSON` (D13 Writes).
5. A new criterion: a test outside fabric builds a pair through hubforge, removes it, and the remove is not refused.

Findings:

- **MEDIUM**, the D13 text "a test of the unset-seam refusal clears the seam through the installer's restore", with D5's "or clears a seam through the installer's restore".
  - **Summary:** a test that builds its hub through hubforge cannot reach an unset seam this way.
  - **Evidence:**
    - A restore puts back the value that was installed before, which is how `InstallWorkspaceContent` swaps and restores at `hubworkspace.go:115-118`.
    - Here the install is hubforge's own, made inside the hub build, and the record never gives the test that restore.
    - If the test calls the installer again and restores, it gets back hubforge's installed seams, not an unset one.
    - The refusal criterion, "A test shows a verb run with a required seam unset refuses", needs a verb to run on a hub, and fabric's tier1 hubs come only from hubforge. So the record names no path from that hub to an unset seam.
  - **Fix:** say how the test reaches it, for example a hubforge option that builds without installing, or an uninstall call whose restore puts hubforge's seams back.

- **MEDIUM**, the D13 text "`internal/hubforge` … calls it when it builds a hub … and returns a restore".
  - **Summary:** the record doesn't say whether hubforge restores the process-global seams it installs. If it restores them in a test cleanup, parallel tests unset each other's seams in the middle of a run.
  - **Evidence:**
    - The seams are process-global swaps.
    - 261 test files that use hubforge also call `t.Parallel()`.
    - Suppose test A's cleanup restores the state from before A's install while test B is still running verbs. B's required seams go back to unset or stale, and B meets the refusal D13 promised it would not.
    - The added criterion, a remove that is not refused, would then flake.
  - **Fix:** state that hubforge installs once per process and never restores, which is idempotent, or say how it combines with parallel tests.

- **No finding** for items 3, 4 and 5. Each is reachable:
  - `internal/ideengine`'s dependencies don't include beamcli or machinecli, so those tests can import the IDE builder.
  - The adapter's `state.WriteJSON` takes the same `StatusLock`.
  - The removal criterion is reachable once hubforge's install is settled (finding 2).

Tool calls:
- grep counting the hubforge-using test files that call `t.Parallel()`. My context had no measure of how often those tests run in parallel.
