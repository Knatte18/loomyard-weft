```yaml
lens: leanness
outcome: reported
```

Leanness review of the adopt-redesign decision record and support log: content to move or cut.

- **MEDIUM — `## Scope` (In list):** every bullet restates a Decision's heading with its D-number, so the list adds nothing beyond the Decisions headings. It is a second place to keep in sync. Evidence: "Synchronous push of each run transition before the run acts (D2)", "A machine registry in Bolt… (D5)", and so on through D11. Fix: cut the list down to scope lines that no Decision states, or point to the Decisions. Out-of-scope items such as the stop button, launchers, account choice, non-orch sessions and re-parent are not in any Decision, so they stay.
- **MEDIUM — `## Open risks`, last bullet:** "The design is large; raddle splits it… the board order of `adopt-revision` names no task yet" is a process note, not a risk. Fix: drop it, or fold it into `## Notes for the plan writer`, which already holds the split.
- **MEDIUM — `## Open risks`, the D10 resume bullet, and the `Review:` acceptance criterion:** both leave the fact D10 rests on (Claude Code resumes a transcript copied into another path's project directory) to a manual check after landing. That is a deferred-research question under the rubric. The leanness part: the bullet restates evidence D10's "Facts this rests on" already lists. The design part is for another lens.
- **LOW — `## Auto-mode assumptions`:** each line restates a pick already stated as a Decision. Examples: "No run acts while origin is unreachable (D2)", the D6 push and removal picks, "lym adopt --release is the escape… (D8)". The support log's question ledger also restates them. Fix: keep only the assumptions no Decision states outright, which are the 10-minute and 24-hour values, and cite the rest by D-number.
- **LOW — `## Constraints`:** several bullets repeat Decisions or the PATTERN changes section. `PATTERN-pair-remote` repeats D2 and its PATTERN change. `PATTERN-dobby-runs` repeats D7, D8 and the change bullet. `PATTERN-owner-machine` repeats D5 and D6. `PATTERN-cli-cobra` ("`--release` is a flag of the existing cobra command") carries nothing a plan writer would get wrong. Fix: keep only the parts that bind the plan writer and are stated nowhere else, such as Finalize finishing on either machine, the code branch going along, and tolerant reads.
- **LOW — `## Goal`, second paragraph:** it lists the bugs settled (A1–A14, A6/A7, A8, A14), which repeats the board entry and the Rationale lines of D5, D9 and D11. Fix: keep one sentence that points to the state map.
- **LOW — D1, last paragraph:** it lists the current consumers ("the dobby's passes, batten's halted reading, the board's layer…, the adopt run report, tender and `lym beam status`"). That is an enumerated consumer list that goes stale, and the "Every actor asks it" sentence just before it is another. The rule "every reader of a run's state takes it from `internal/runfsm`" is enough. Fix: state the rule and let Plan find the readers.
- **LOW — D3 cache list:** it is long, but it passes the completeness test. The plan writer needs to know which records become caches, so keep it. No change.
- **LOW — support log, Turn 2:** fine where it is, since the support log is the place for exploration narrative. No relocation is needed.

No further tool calls made.
