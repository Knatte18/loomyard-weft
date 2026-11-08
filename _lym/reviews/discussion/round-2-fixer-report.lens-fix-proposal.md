```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged each fix that carried out a reviewer proposal as written (F1 to F10). F2, F4 to F10: no case found.

**[MEDIUM] F1: a failed-build empty map blocks retry and is judged against silently**
- Changed text: D5, "A package whose coverage build fails … gets a map recording its Go file hashes and no tests … stale again only when its files change or its map passes `test_map_max_age_days`".
- Case 1: a package's coverage build can fail for a reason outside its own files:
  - a dependency's compile error at that HEAD;
  - the new `redundancy_build_timeout_min` firing under load ("counts as failed").
  The package's files do not change when the cause clears, so the empty map stands for up to 7 days.
- Case 2: D7 has no outcome for such a map.
  Step 6 marks only "a package with no map" as not judged, so D7 judges a package with an empty failure map against zero map tests.
  Its new tests are reported unique, and nothing on the line says the map holds a build failure.
- Case 3: the repeat skip at D7 line 137 retries only on "an error line, a package with no map or a test past the budget".
  An empty failure map does not trigger a retry.
- Evidence: D5's new paragraph; D7 step 6, "A package with no map is reported as not judged".
- Sound alternative:
  - D7 reports a map that records a build failure as not judged, with the cause, and its report never skips the next call.
  - D5 keeps F1's anti-loop property by not rebuilding the package within the call or batch that failed, rather than until its files change.

**[LOW] F3: hashing every package that holds a dropped test makes the report stale too often**
- Changed text: D7 step 6, "and the hash of each Go file of every package holding one". Also D8, "every recorded package's Go files hash the same".
- Case: a diff changes a `tier2` test in package X and nothing judged in X.
  A later fix commit edits a non-test file in X.
  The step 1 set and its function hashes are unchanged, and no verdict depends on X's files.
  Still, the file-hash check fails, so Code-Burler-Review gets the no-report line and the repeat gate call rebuilds and reruns.
- Evidence: the new step 6 text and D8 match text above.
- Sound alternative: record function hashes for every step 1 test, but Go file hashes only for judged packages, which was the earlier wording.

Outside my lens: `internal/runproc/doc.go:44` says a session end that is not a teardown uses `EndSparing` to spare `RoleDobby`. A hub-owner `EndSparing` would end the `test-map` record, so the refresh would die while the dobby lives on. The fix did not change this; it comes from the existing registration under `runproc.HubOwner`.

Tool calls:
- One grep of `cmd/testtiming` for spawn-scan callers, to check F9's move. My context had only the directory listing; the only caller is `redundancy.go:449`, so the move breaks nothing.
- The same grep across `internal` for `RoleDobby` stop routes, to check F2's stop route against other dobby-ending paths. My context held only `orchcli/cli.go:276` and `runproc/end.go`.
