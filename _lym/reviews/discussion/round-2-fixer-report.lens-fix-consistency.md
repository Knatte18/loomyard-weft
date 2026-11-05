```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-2 fix set agrees with its sibling decisions, criteria and cross-references, the fix-consistency lens.

- **MEDIUM**: the D3 step 3 change ("A held message with that hub path and id that differs in any of those is an id clash, such as a hub re-created at the same path restarting its ids") contradicts Constraints first bullet, which still says "ids per hub that are never reused". Finding F6 named "Constraints first bullet" in its location, but that bullet was left as it was. The record now says in one place that ids are never reused, and in another that a re-created hub restarts them. Evidence: decision-record.md Constraints first bullet; review F6 `location: ... D3 step 3, Constraints first bullet`.
- **LOW**: the Open risks bullet and the Q6 ledger entry now treat dropping unconfirmed mail as settled by the operator, but `## Auto-mode assumptions` still lists "A confirm that answers `no` drops the message instead of showing it as unverified (D3)" as an auto-pick. That twin was not updated to say the operator's added decision settled it. Evidence: decision-record.md Auto-mode assumptions fourth bullet; support-log.md Q6 "settled by the operator's added decision, 2026-11-05".
- **LOW**: the new D3 coalescing bound says "the first after the hour counts the drops journaled since the last". It does not say whether that notice goes out on the next drop or on a timer. If a timer sends it, that timer is a new daemon wakeup, and the `PATTERN-daemon-wakeups` constraint bullet names only D2's outbound worker for the declared list in `cmd/lym/daemonwakeups_test.go`. The constraint and the bound disagree, or the bound is ambiguous. Evidence: D3 second new Bound; Constraints `PATTERN-daemon-wakeups` bullet.

No further tool calls made.
