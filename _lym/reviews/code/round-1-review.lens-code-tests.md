```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test row that pins it, or none.

## Behaviour → pinning test

1. **An open BLOCKING ledger entry at a spent budget escalates with cause `budget`, and the detail names the keys (`blockingEscalationDetail`, `openBlockingKeys`).** Pinned by the `TestBouncer_AutoAccept` row "BlockingEscalates escalates an open BLOCKING entry to the parent instead of accepting" (`bouncer_escalation_test.go:472`). It asserts Awaiting, the cause `budget` and the detail text `BLOCKING findings open (hard)`.
2. **The same escalation with auto-accept off, so it runs ahead of the soft-cap rule.** Pinned by the row at `bouncer_escalation_test.go:475`.
3. **Below the budget, an open BLOCKING entry is left to the next round, with no escalation.** Pinned by the row at `:478`. It asserts Stuck, and the harness's `requireNoEscalation` checks that nothing was escalated.
4. **A parent continue recorded over a BlockingEscalates escalation runs a budget-exempt round.** Pinned by the row at `:481`, through `wantExempt`.
5. **BlockingEscalates with no open BLOCKING entry still auto-accepts and carries the open findings.** Pinned by the existing row at `:469`.
6. **A ledger that cannot be read escalates with cause `budget` and the "could not be read" detail (the `err != nil` branch of `blockingEscalationDetail`, `bouncer.go:1550–1552`).** None. No test row has an unreadable or malformed round ledger with BlockingEscalates set. The grep of `internal/shedadapters/*_test.go` found no such row.
7. **A window opened by the hard cap alone (round at or past the hard cap, soft cap not reached) escalates with cause `budget`, not `hard-cap`.** Only weakly pinned. The `entries_bouncer_test.go:1431` row "a hard cap of 1 offers only accept", with `blocking_escalates`, asserts Awaiting but not the cause. Before the change that row reached Awaiting through cause `hard-cap`, so a regression that sends the round to `hard-cap` again still passes it.
8. **The recipe key `blocking_escalates` is wired from the shedrecipe Bouncer entry to `BouncerConfig`.** Pinned by `TestBouncerEntry_BlockingEscalatesAndColorSegment` (`entries_bouncer_test.go:1430`), which now expects Awaiting.
9. **`CheckReviewRound` admits a new BLOCKING `decision`-class `origin: focus` finding outside the changed files.** Pinned by `NewFocusDecisionOnAnUntouchedFilePasses` (`roundcheck_test.go:274`).
10. **An `origin: focus` finding of another class gets no exemption.** Pinned by `NewFocusFindingOfAnotherClassOnAnUntouchedFileIsRefused` (`:275`).
11. **An `origin: focus` decision finding below BLOCKING is refused.** Pinned by `NewFocusDecisionBelowBlockingIsRefused` (`:276`).
12. **The recipe switches: `escalate_decisions`, `decisions_escalated` and `blocking_escalates` on the Code, Harden, Discussion and Raddle rows across tapestry, darn, sampler, arras and hardener.** Pinned by `TestDecisionSwitches_SetOnTheRowsThatAskTheParent` (`decisionswitches_test.go`).
13. **A Harden-Bouncer round holding an `origin: focus` decision finding escalates with cause `decision`.** Pinned by the switch row in item 12 together with the existing `routeDecisions` tests. The route itself is unchanged.
14. **The template comment, the stencils and the spec text.** Static prose, exempt.

## Findings

- **[LOW] design** — `internal/shedadapters/bouncer.go:1550` (`blockingEscalationDetail`, the unreadable-ledger branch)
  - **Summary:** the diff adds the behaviour "a round whose ledger cannot be read escalates with cause `budget`", which the commit message, the `BouncerConfig.BlockingEscalates` doc and `shed-recipe-spec.md` all state. No test row covers it.
  - **Fix:** add a `TestBouncer_AutoAccept` row with `blockingEscalates: true`, a spent budget and a round-2 ledger that does not parse. Expect Awaiting, cause `budget`, and a detail containing "could not be read".
  - **Evidence:** `bouncer_escalation_test.go:469–483`, where no row has an unreadable ledger. The mutation report lists 2 uncovered changed blocks.
- **[LOW] design** — `internal/shedrecipe/entries_bouncer_test.go:1431` (the row "a hard cap of 1 offers only accept")
  - **Summary:** with `blocking_escalates`, a window opened by the hard cap alone now escalates with cause `budget` instead of `hard-cap`. The only row reaching that path asserts the outcome but not the cause.
  - **Fix:** assert the escalation cause on that row, or add a `TestBouncer_AutoAccept` row with `HardCap` reached and the budget not spent that expects cause `budget`.
  - **Evidence:** `bouncer.go:1431–1437`, where `windowOpen` includes `atHardCap`; `entries_bouncer_test.go:1431`.
- **[NIT] consistency** — `internal/loomrecipe/decisionswitches_test.go` (`func TestDecisionSwitches_SetOnTheRowsThatAskTheParent`)
  - **Summary:** the redundancy report lists the test as contained, covered by `TestAmendmentTarget`. That test asserts only `AmendmentTarget`'s row name for each recipe (`recipe_test.go:300–327`) and never the switch keys, so this test pins what coverage cannot see.
  - **Fix:** mark the test `//testtiming:keep pins which shipped recipe rows set escalate_decisions, decisions_escalated and blocking_escalates` directly above its `func Test…` line.
  - **Evidence:** `recipe_test.go:300–327`; `decisionswitches_test.go`, as the diff shows it.

## Tool calls

1. A grep of `internal/shedadapters/*_test.go` for BlockingEscalates rows and unreadable-ledger cases, plus a read of `TestBouncer_AutoAccept`'s harness (`bouncer_escalation_test.go:440–620`). My context held only the diff hunks of that test, not its harness assertions or the full row set.
