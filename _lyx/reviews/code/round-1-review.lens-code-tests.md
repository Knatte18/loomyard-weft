```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, the test that pins it, or none. I found five gaps, F1 to F5 below. I found no redundant tests.

**Behaviour → pinning test**

| Behaviour | Pinned by |
|---|---|
| Bouncer `decision` escalation: every verdict, before refusal, blocking-without-rejudge, auto-accept, re-call Awaiting, continue/accept, carry-over, held refusal, hard-cap halt, hard-cap accept-only, settled re-entry, replay, non-escalation paths | `TestBouncer_DecisionEscalation` subtests (`internal/shedadapters/bouncer_decision_test.go:199-723`) |
| `escalate_decisions` row key | `internal/shedrecipe/entries_bouncer_test.go:1255-1256` |
| `decisions_escalated` row key | `internal/shedrecipe/entries_burlerhalves_test.go:746-754` |
| Fix row sends no notice and withholds the findings | `burlerfix_test.go` (`DecisionsEscalated: true` at 297) |
| Both keys set together on Discussion rows only | `TestDecisionSwitches_SetTogetherOnLoomsDiscussionRowsOnly` |
| `EscalationDecision.valid` and the circling cause message | `circling_test.go` (+10 lines) |
| Arbiter refuses a `decision` escalation | `arbiterstop_test.go:75` |
| Arbiter refuses a decision halt | `arbiterstop_test.go:30` |
| Carry-over `AddedSince` rendering | `carryover_test.go:62,82` |
| Carry-over `AddedSince` assembly (`addedSinceSnapshot`) | the "ARecordedAcceptCarriesEveryDecisionFinding..." subtest |
| Lenses `review-discussion` fan members | `lenses/config_test.go` (fan table, sixth entry) |
| A hub's own lens text beats the template | `lenses/config_test.go` new subtest |
| `RenameProducers` undecodable status file | `shedengine/rename_test.go` new subtest |
| Gate key rename `merge-in` → `parent-align` | `gateslot/config_test.go:562-564` |
| Row rename `Merge-In` → `Parent-Align` in reports | `reportengine/timing_test.go` (`retiredMergeInRow` rows) |
| Retired loom keys dropped by `MigrateConfig` | `loomengine/config_test.go` (`alignment_pr_approve` row at 1233) |
| Retired loom keys handled at read time in run settings | `configreg/runsettings_test.go` |
| `merge_fix_attempts` → `parent_align_fix_attempts` | `landingshed/config_test.go` (`TestMigrateConfig`) |
| `DecisionNoteTitle` in the orch role | `orchengine/prompt_test.go:41` |
| Darn no-record refusal text | `loomcli/decision_test.go:385` |
| Stencil prose for D7, D9, D10 | none, and none expected (static prose is exempt) |
| Archived record decoding in reportengine (D4) | indirect only, via fixtures (`fake_test.go:328`, `:408`; `timing_test.go:76`) |
| Parent-Align reads the old `Merge-In` fixer attempt record and removes it with its own | none |
| Code-Perch reads the old row-named fixer report when the new one is absent | none |
| Resume of a status parked on `Merge-In` | rename mechanism only, `rename_test.go` |
| Run profile or `run_settings` holding `merge_fix_attempts` or `merge-in` is migrated with a Warn | none |

**Findings**

**F1 — MEDIUM, scope.** Location: `internal/landingshed/parentalign.go` (the `FixerAttemptsPath` fallback), tests in `parentalign_fixer_test.go` and `fixerattempts_test.go`.
- Behaviour with no coverage: the `Parent-Align` row reads the old `Merge-In-fixer-attempts.json` when its own record is absent, and removes the old record wherever it removes its own.
- Why it matters: an in-flight run keeps its spent attempts, `Baseline`, `AlignFrom` and `Halt` only through this path, and the decision record states it as acceptance criterion A254.
- What I saw: my greps found only `FixerAttemptsPath(..., "merge-in")` calls, with no old-name fallback assertion.
- I did not open the whole of `parentalign_fixer_test.go`, so confirm before fixing.

**F2 — MEDIUM, scope.** Location: the Code-Perch review-input reader, which I could not locate in the diff stat; a test belongs beside it.
- Behaviour with no coverage: the reader takes the old row-named fixer report when the new one is absent. Criterion A253 requires "a run past the row at upgrade still gives the Code-Perch its latest fixer report".
- What I saw: no test names the old report.

**F3 — LOW, scope.** Location: `internal/reportengine/archivedrecords.go:1`, with no `archivedrecords_test.go`.
- Behaviour with no direct coverage: the read-only decoders for `request.json`, `delivery.json`, `verdict.json`, `alignment.json`, `attempt.json` and `verdict-<token>.yaml`.
- Existing coverage: the fixtures in `fake_test.go` and `timing_test.go` feed some shapes, through the timing and report tests.
- Gap: I saw no fixture for `attempt.json` or `delivery.json`. A decoder drift there would pass silently, and criterion A236 depends on all of them.

**F4 — LOW, scope.** Location: `internal/configreg/runsettings_test.go`.
- Behaviour with no coverage: read-time migration of a profile or `run_settings` holding `merge_fix_attempts` or the `merge-in` gate key, with a Warn.
- What I saw: the diff touches `runsettings_test.go`, and the only retired-key hits in the grep were the alignment and parent-review keys. Criterion A255 names the reconcile case, which `gateslot` and `landingshed` `TestMigrateConfig` cover. The read-time case for the `merge-in` gate key and `merge_fix_attempts` is not visible.

**F5 — LOW, scope.** Location: the status-vs-carry-across test for `Parent-Align`.
- Behaviour with no coverage: `loomrecipe`'s retired-rows list including `Merge-In` → `Parent-Align`.
- What I saw: `shedengine/rename_test.go` pins the generic mechanism, and `peers_test.go:109` uses `retired-row-peer` with `Parent-Align`. I found no recipe-level test that a `Merge-In` status resumes on `Parent-Align`, which is criterion A253.

**Test economy**
- I saw no duplicate coverage and no per-helper test.
- I saw no assertion on a stencil's static prose.
- `rename_test.go`'s new "undecodable status file" subtest covers the final commit's added behaviour. It is not a duplicate.

**Tool calls (5 of 6 used)**
1. Bash grep of `Test*` names across the decision, arbiter, circling, burlerfix, carryover, lenses, rename, landingshed and reportengine tests: my context had no test inventory.
2. Bash `git diff --stat` of `*_test.go` plus a listing of the reportengine files: to see which tests changed and whether `archivedrecords_test.go` exists.
3. Bash grep of subtest names in `bouncer_decision_test.go` plus the diffs of `timing_test.go`, `lenses/config_test.go` and `rename_test.go`: to map the Bouncer behaviours and check the other three diffs.
4. Bash grep of the archived-record, retired-key, `Parent-Align` and `merge-in` fixer symbols across the reportengine, `gateslot`, `configreg`, `loomengine` and `landingshed` tests: to look for coverage of the carry-across and migration behaviours.
5. Bash grep for the old-name fallback, arbiter decision cases, `Parent-Align` rename coverage and `DecisionNoteTitle` tests: to settle F1, F2 and F5 and the arbiter rows.
