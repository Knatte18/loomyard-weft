---
round: 1
focus:
  - "Card 1 `Edit` targets: `internal/boardcli/doc.go`, `plugins/ly/skills/board/SKILL.md` and `internal/boardcli/intake_test.go` are file paths while the Intent names specific edits; settle whether each file-path target needs member glyphs, and whether `Board.boardCriticalSection` needs the re-sign arrow given the Intent changes its flow rather than its signature."
  - "Card 1 sizing: one card creates the lock symbols, edits the critical section, three docs files, the intake test and the budget file; settle whether the granularity rule wants the docs or the intake-test row split out, or whether they are one reviewable unit."
  - "Test economy across cards: Card 1's new `TestRunLock` rows for `RunStatus`/`IsRunStatus` and the `intake_test.go` row, Card 2's `TestSetRunStatus` locked-entry row, and Card 3's table; settle whether any row repeats behavior an existing test or another new test already covers, naming that test."
  - "Card order and `Uses`: Card 2's `TestSetRunStatus` locked-entry row relies on the Card 1 lock and Card 3 relies on Card 2's `OpenHub` and `SetRunStatus`; settle that no card depends on a later card, and that Card 2's `Delete` of `openHubBoard` leaves no caller outside the two edited ones."
  - "Attack-surface bounds: the Intents of Card 1 (unguarded `set-status`, prune exception) and Card 3 (a failed claim write only warns, the run proceeds) each state a bound; settle whether each bound is stated well enough, including what the warn-and-proceed path lets through."
  - "Fidelity to the decision record: every Decision and Constraint is carried by a card, including the refusal text's two ways forward (Decision 5), the batch listing one slug twice, the intake-import ordering before the GitHub comment, and Decision 10 (no PATTERN entry); and no card adds scope the record does not license, such as `SetRunStatus` replacing a locked entry's status or `OpenHub` as new public surface."
  - "Verify section: confirm the command list covers every package the cards target (`internal/boardengine`, `internal/boardcli`, `internal/loomcli`, `internal/battencli`) and follows the order the plan template states."
  - "ImpactSummary on the `Edit` and `Delete` cards: settle that each states a real blast-radius conclusion rather than restating Intent, in particular Card 2's `Delete` of `openHubBoard` and Card 1's claim about every mutating Board method."
---

Seed focus from reading the plan and the decision record.
No entry restates a `Do not flag` item: dependency completeness, `ImpactSummary` on non-`Edit`/`Delete` cards and the mechanical spec checks are left out.
