```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the F1 fix covers its whole set (the skip-bound cases).

F1's premise is a set of bounds: a predicate that skips only `docs/research/<report>/data/...`. I ran its search once, on the predicate and the subtest's fixture.

Findings: none.

The fixed subtest now covers these members:
- Skipped: `docs/research/x/data/prompts/step.md`.
- Scanned: `docs/research/x/report.md`.
- Scanned, so a skip widened to any `data/` under `docs/` fails: `docs/other/data/x.md`.
- Scanned, so a skip widened to any depth under a report fails: `docs/research/x/sub/data/x.md`.

The review's other probes (the four-part `docs/research/x/data` and `docs/research/data/a.md`) are not rows in the subtest. The review's `fix` key did not ask for them, and it called the predicate correct on them.

Unchecked: nothing, since F1 is the only finding.

Tool calls: 0 further calls. My context already held the changed subtest, the review and the gate-green output, so I read nothing more.
