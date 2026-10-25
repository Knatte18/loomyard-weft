```yaml
lens: code-tests
outcome: reported
```

Task: own the tests of the diff (lens code-tests), applying the "Test economy" rule.

**Per behaviour, the test that pins it:**
- `docs/overview.md` merge, the only content in this amendment's scope: no behaviour and no test. A doc has nothing for a test to pin, and the rubric's "Test economy" has no case for a doc-only change. Not applicable.
- The beam-window code and test changes (battencli `overview*`, beamcli, orchcli `runs`/`start`, reedengine `windoworder`): out of scope. They are the settled remainder, and the amendment rule says to report nothing about it. The changes since the settle are only the Parent-Align result for `docs/overview.md`, so I did not check whether tests pin these behaviours.

**Findings:** none.

**Tool calls:** none. My context already covers the `docs/overview.md` diffs and the amendment note, and no test file is in scope.
