```yaml
lens: fix-proposal
outcome: reported
```

Challenged proposals: F1, F2, F5 and F14. Two findings, both on F1.

**MEDIUM — Bouncer judge can time out while waiting for the parent (F1 proposal).**
- Changed text: `internal/shedadapters/bouncer.go` `judgeCall`, which appends `judgeParentBlock` to the prompt, and `parentseatwiring.go` `baseParentSeat` `Wait`.
- The ask deadline comes from `CapParentReviewWait`, which is capped against `review_timeout_min` (240 in `internal/loomengine/template.yaml:35`) minus allowances. The judge spec in `judgeCall` sets no `Timeout`, so it runs under shuttle's `run_timeout_min`, which is 30 (`internal/shuttleengine/template.yaml:4`).
- With the defaults, `code_review_parent_timeout_min` is 30 and the cap leaves about 190 minutes of room, so the deadline stays at 30 minutes after the ask.
- The judge session is then spawned after the ask and told to `await` that parent. It hits its 30-minute run timeout before the deadline, and `judgeCall` degrades with "judge run did not complete".
- Drift's F5 fix capped the wait by the judge's own timeout. The Bouncer path carries the same defect, since the proposal only said to cap "as CapParentReviewWait caps it", which bounds against the wrong timeout.

**MEDIUM — Prose says parent findings rule out convergence, but nothing enforces it (F1 proposal).**
- Changed text: `judgeParentBlock` ("Such a report rules out `CONVERGED`"), `orch-role.md` ("such a round is not converged, and the next round's reviewer carries them") and `docs/overview.md` ("keeps every finding the parent returns").
- The Go code adds only a prompt block. The judge's verdict and ledger parsers know no parent report, and the Bouncer's settle does not read it.
- A judge that writes `CONVERGED` ignores the parent's findings silently. On the last round, at the hard cap or an accept, there is no next reviewer, so the parent's findings are never fixed.
- The earlier design had a coverage gate (`ParentCoverageGateEntry`) holding the review to every report id, and this checkpoint has no equivalent.
- The docs state a behaviour the code does not hold.

No finding on F2, F5, F14 and the test-only findings (F6–F13). Checked cases:
- F2: a findings answer followed by a rewrite gives a second ask; 6 attempts cover two ask cycles.
- F5: `judgeOpenedAt` from the session mark only makes the limit more conservative on resume.
- F14: legacy untagged asks cost one extra ask at upgrade.

Tool calls: 1 (`grep` of the `review_timeout_min` and `run_timeout_min` defaults; my context lacked the template and shuttle config values).
