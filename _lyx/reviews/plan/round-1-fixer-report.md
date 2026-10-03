# Fixer report — plan, round 1

Every finding in `round-1-review.md` is fixed in the plan cards; nothing is deferred.

## Fixed

- **F1 (MEDIUM)** — `11-anomalies-under-step.md`: the entry observation moves into a new `(c *loomCLI) observeStepEntry()` that `loomPreStepUnmarked` calls last, after the busy probe, the bootstrap and `reed.Up()` succeed.
  The card states why: a `PreStep` refusal no longer spends the clean-handoff marker, and for a seeded run the bootstrap leaves the status file untouched, so the observation reads the same state.
- **F2 (MEDIUM)** — `11-anomalies-under-step.md`: a new `halt_test.go` case drives the generic step body twice with `PreStep` calling `c.observeStepEntry()`, over a shed whose first row returns Done and leaves the run `running`, the status seeded once.
  The second step files no `crash-resume`; with the marker removed between the steps it files one.
- **F3 (LOW)** — `11-anomalies-under-step.md`: the bound paragraph names the residual `crash-resume` after a `BuildShed` refusal or a `shed.Step` error that leaves the run `running`, the same one `run` has after the same failures.
- **F4 (LOW)** — `10-anomaly-pending-retry.md`: the test list adds the unreadable-status and undecodable-product arms, the cancelled context with no crash-resume (pending untouched, no filer), and the cancelled context with a crash-resume (files it and retries the pending entries).
- **F5 (LOW)** — `10-anomaly-pending-retry.md`: `internal/loomcli/selfreport_github_test.go` joins the Edit list, and the filing-failure test's doc and assertions cover the pending entry.
- **F6 (LOW)** — `11-anomalies-under-step.md` and `12-halt-filing-ownership.md`: both bounds name the residual.
  A halt whose Tier 1 filing failed is retried from the pending list on the next pass, and one whose pass was skipped is detected again only while the run is still halted.
  Neither is filed if no later `run` or `step` runs in that worktree.
- **F7 (NIT)** — card 11's halt-note line now reads `Tier 1 anomaly filing files the halt event itself; file only the lyx problems behind it.`, and card 12 quotes it verbatim.

## Deferred

None.
