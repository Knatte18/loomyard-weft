# Lens review

All four forks returned reports.

- forced-edits, BLOCKING, card 8: `auditedNegativeVerdictReturns` still keys `removeStrandIfLive [Errorf]`, so the completion-signal tripwire fails on card 8 — fixed: card 8 edits the audited map and its doc bullet to `removeStrandIfPresent`.
- forced-edits, MEDIUM, card 1: lists `render/types.go` as a file while its Intent changes members — fixed: lists `render#Display`, `render#Anchor` and `render#AnchorOwnWindow`.
- forced-edits, MEDIUM, card 7: lists `bar.go` as a file while its Intent changes `statusFormatButtons` — fixed: lists the member. The empty `Create` heads for the two new formats are kept and the Intent says the heads only declare names (LOW, a head is a declaration).
- forced-edits, LOW, card 10: `seatSpec`'s doc sentence must be replaced whole — fixed: the card says so.
- forced-edits, LOW, `docs/overview.md:649`'s display vocabulary — rejected: the sentence stays true, and the own-window vocabulary is documented in reed's doc.go and the reed bullet.
- forced-edits, LOW, card 13: the shuttle bullet is duplicated on `docs/overview.md:375` — fixed: card 13 drops the older copy and edits the one left.
- forced-edits, LOW, cards 4 and 6 both touch `killStrandPanes` — rejected: a note, both edits are listed and ordered.
- row-reachability, MEDIUM, card 3: `TestOpenWindow`'s four-field fixtures break under the five-field format — fixed: card 3 rewrites them with the own flag `0`.
- row-reachability, MEDIUM, card 13: a cobra-required `--role` cannot produce the asserted JSON error — fixed: `RunE` refuses an empty `--role` with a JSON error, `Use` is `open`, and `--model` is parsed before any config read.
- row-reachability, LOW, card 3: `TestRepairNames_SessionName` lacks the fields its new row needs — fixed: the card names the `ownWindow` and `windowPanes` row fields and the re-scripted listing.
- row-reachability, LOW, card 8: `cleanupDiedCaptureKept` had no assertion branch — fixed: the row loop's switch gains its case.
- row-reachability, LOW, card 8: the step-stop test's exact assertions must change — fixed: the card names the new strands, seeds and expected report.
- criteria-coverage, MEDIUM, card 5: the group-window split guard (D7) had no row — fixed: new `TestJoinGroupWindowLocked_RefusesASilentlyReusedPane`, budgeted.
- criteria-coverage, MEDIUM, card 5: the join's split took focus in the group window — fixed: `split-window -d`, with the group subtest asserting the active pane and current window unchanged.
- criteria-coverage, MEDIUM, card 5: a dead pane counting toward the live count was untested — fixed: subtest `ADeadMemberStillCountsTowardTheGroup`.
- criteria-coverage, MEDIUM, cards 8 and 10: the handle-form stop of a dead own-window advisor was untested — fixed: rows in `TestStop_RecordsTheStopBeforeRemovingTheStrand`, which runs the handle form `join` uses.
- criteria-coverage, MEDIUM, card 13: the self-removal's success path and `explore-2` — fixed in part: a row pins that two opens pass the role with no name override. The rest is rejected with citations: numbering is pinned by `internal/reedengine#TestStrandNameLocked_Names`, removal by full name by `internal/reedcli#TestStrandNamingAndList` (`remove --name <full name>`, `naming_integration_test.go`), and the window end on removal by card 4's `RemoveEndsTheOwnWindow`. A real `lyx` binary in a hermetic pane is not available to a tmux-tier test.
- criteria-coverage, LOW, card 3: a group window whose first strand is mid-add is argued, not tested — rejected: `planReconcile` reads the strand table it is handed, and `OwnWindowHoldingABoundPaneIsKept` pins that a bound pane keeps its window.
- criteria-coverage, LOW, card 13: `PATTERN-cliwire-sole-wiring` — fixed: card 13 records why the verb takes the shuttle group's existing wiring.
- criteria-coverage, LOW, card 3: the select falls back to the current window with no window 0 pane recorded — rejected: up and resume always ensure Selvage, so the fallback is reached only on a session with no Selvage, where window 0 cannot be told apart anyway.
- criteria-coverage, LOW, card 2: the add-time check also blocks window 0 adds — fixed: card 2 states the scope.
- test-economy, LOW, card 8: the died window 0 row repeats `dead pane is died and keeps the strand` — fixed: dropped.
- test-economy, LOW, card 3: the own-window resume-plan rows reach no new branch — fixed: dropped.
- test-economy, LOW, card 6: the failed-self-removal subtest repeats card 4 subtests — fixed: the self-written marker is folded into card 4's Resume subtest, and the card 6 subtest is dropped.
- test-economy, LOW, card 6: `GroupRecordsAfreshAfterItsLastStrandGoes` is covered by pure tests — fixed: dropped.
- test-economy, LOW, card 13: `TestOpenCmd_FlagValidation` should be rows of `TestRunCLI_Run_FlagValidation` — fixed: rows with a command-builder field, and the `shuttlecli` budget raise dropped.
