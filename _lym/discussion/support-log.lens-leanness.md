```yaml
lens: leanness
outcome: reported
```

Task: review the decision record and support log through the leanness lens, looking for content to move or cut.

**Findings**

1. `MEDIUM` — `## Auto-mode assumptions`: the section restates decisions the record already makes.
   - Evidence: these bullets repeat their decisions: "uses the round's own lens fan" (D2), "timeout key … 60 minutes" (D3), "`config.toml` is rendered whole" (D5), "deny notice and passages ride the prompt file" (D6), "Skills load as files read by path" (D9) and "Codex on Windows is refused" (D13). The support log's question ledger already marks each one as an auto-pick.
   - Fix: keep only the assumptions no decision states, the Burler-review-only scope and the no-creel-delivery choice, or cut each bullet down to a pointer to its decision.

2. `MEDIUM` — `## Constraints` against `## PATTERN changes`: the record contradicts itself.
   - Evidence: Constraints lists `PATTERN-fs-links` among the entries that "hold", while PATTERN changes changes that entry and D13 adds `CreateFileLink`.
   - Fix: drop `PATTERN-fs-links` from the "hold" bullet. The PATTERN changes bullet already covers it.

3. `LOW` — `## Constraints`: one bullet repeats a decision.
   - Evidence: "Codex efforts above `high` are not used (operator, 2026-10-27)" repeats D4's rationale and its `efforts` default.
   - Fix: say it once, in D4.

4. `LOW` — D9 against `## Open risks`: one fact appears in both places.
   - Evidence: D9's last line, "This replaces the board step's 'skills sync into the Codex home'", and the first Open risks bullet state the same departure.
   - Fix: keep it only in Open risks, where the parent sees it.

5. `LOW` — D6's bound against D2's bound: both say the same thing.
   - Evidence: D6 says "`workspace-write` lets the session write anywhere in the worktree … nothing else stops a write but the prompt", and D2 says "it writes inside the worktree, and its prompt alone keeps it off target files".
   - Fix: keep it in one place, D6, and have D2 refer to it.

6. `LOW` — `## Open risks`: the last bullet partly restates D5.
   - Evidence: "a release that writes more is overwritten at the next launch, and the trust check after startup warns" repeats D5's bound and its Warn check.
   - Fix: keep only the new part, that the `[tui]` set rests on one spike.

7. `LOW` — `## Notes for the plan writer`: this is plan-level detail, but the spec allows a hint to name a symbol, so I am not asking for a cut.
   - Evidence: it names the tests `TestNames_EqualModelspecEngineNames` and `TestToldGeometryInvariant_AllowlistOnly`.
   - Note only: no action needed.

8. `LOW` — D2 is long, and two lines carry no design weight.
   - Evidence: "`ReviewResult` names the assistant's outcome and copied report path; no decision reads them" is a result-struct detail. "The chair's step-2 rules gain one part, rendered only in an assisted round, for solo and fanned rounds alike" partly repeats D1's wait rule.
   - Fix: cut the `ReviewResult` line and point the wait at D1.

The cuts above cost the plan writer no requirement: each one's content is already stated in its decision.

Further tool calls: none.
