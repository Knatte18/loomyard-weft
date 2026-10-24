```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, applying test economy to the behaviours this round's amendment changes.

**Per behaviour, the test that pins it:**
- The amendment names only `docs/overview.md`, which is prose that no build or test reads. I found no behaviour it adds or changes, so no test is owed and none is missing.
- The settled profile code under `internal/runprofile`, `internal/boardcli`, `internal/configreg`, `internal/battenshed` and `internal/battencli` is outside this round. The parent changed none of those paths since the settle: the stat of the settle tip against `c9c4e7dd` for them was empty. No test row there became stale or redundant because of the merge.
- I did not re-check the settled tests for redundancy, since the focus file puts the settled remainder out of scope.

**Findings:** none.

**Tool calls:** none. My context already held everything this lens needed.
