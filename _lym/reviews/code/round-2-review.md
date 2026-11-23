---
verdict: BLOCKING
findings:
  - id: F13
    severity: BLOCKING
    class: design
    location: internal/shedadapters/bouncerdecision.go#escalateUndecided
    summary: "Outside the soft-cap window, a CONTINUE or CIRCLING round whose review holds a decision finding and whose ledger holds an open BLOCKING entry of another class escalates with cause decision, which the arbiter may settle. A recorded accept then reaches settleDecisionAccept, where decisionRefusal for a non-CONVERGED verdict runs only the dropped-key check, so acceptRound passes the round with the non-decision BLOCKING entry still open and without the parent. Before this diff the Code-Review and Harden-Review rows had no decision escalation, so such a round could only continue. Now a Code-Review round below its budget, and a hardener round at any budget (Harden-Bouncer sets no blocking_escalates), lands with an open BLOCKING finding on an arbiter accept that was meant to rule only on the decision or focus findings. That is against goal 2 and against goal 3's accept with a stated reason per undriven entry."
    fix: "In internal/shedadapters/bouncerdecision.go, make an accept recorded on a decision escalation rule only on the decision-class findings. In settleDecisionAccept, before acceptRound, when the verdict is CONTINUE or CIRCLING, call openBlockingKeys(round, burlerengine.ClassDecision). When it returns keys or an error: outside the soft-cap window (softCapWindowOpen false), run the next round the way settleDecisionContinue does (ensureFocus(round + 1), return Stuck with BudgetExempt set when budgetReached), so the fixer fixes the open entries. Inside the window, escalate with cause budget through escalateWith, with a detail naming the open keys, so only the parent accepts. Name this rule in the doc comments of routeDecisions and settleDecisionAccept. In TestBouncer_AutoAccept (internal/shedadapters/bouncer_escalation_test.go), add a row with escalateDecisions set, blockingEscalates unset, bounces belowBounces, the decision report decisionReviewContent(D1), a ledger holding alpha resolved and hard open scope BLOCKING, and decide CirclingAccept. It expects Stuck, wantSettled false and the judge outputs kept. Add a sibling row at spentBounces that expects Awaiting with cause EscalationBudget."
  - id: F2
    severity: BLOCKING
    class: design
    carries: F2
    location: internal/loomengine/reinterpret.go#FlagCodeAutoAccept
    summary: "The loom reinterpretation flags only a hub loom.yaml that sets code_auto_accept to true. The template default is true, and PATTERN-config-layers keeps a hub base file sparse, holding only tuned units. So a hub that takes the default, which is the common case, holds no such key, and neither does a hub with no loom.yaml. Those hubs get the same meaning change with no flag, against the rule that a meaning change ships a reinterpretation that rewrites or flags the value. TestFlagCodeAutoAccept's OtherKeysOnly and HeaderOnly rows pin the missing flag. The rest of the prior F2 also stands: _lym/landing/summary.md still calls the door two-way, which the rubric rules out for a change to what a config setting means. Its blast radius of shed alone also leaves out the burler round check behind lym burler validate and the hub reconcile notice, both of which the diff changes."
    fix: "In internal/loomengine/reinterpret.go, FlagCodeAutoAccept flags a file that sets no code_auto_accept as well as one that sets it to true, with the same way-forward line. It accepts nil bytes (absent file) and flags that case too. It leaves the bytes unchanged and flags only a file setting false. Update its doc comment to match. In internal/configreg/configreg.go, set RunsWithoutFile: true on the loom Reinterpretation entry so the walk runs it when loom.yaml is absent. In internal/loomengine/reinterpret_test.go, change the OtherKeysOnly and HeaderOnly rows to wantFlagged true and add a nil-input row that expects one flag. In _lym/landing/summary.md under ## Merge danger, change Door to one-way. Widen Blast radius to name the burler review round check and the hub config reconcile alongside shed."
  - id: F11
    severity: BLOCKING
    class: design
    origin: assist:assist
    location: internal/shedrecipe/entries_burler.go:183
    summary: "The hardener-only focus permission is set at two sites that no test asserts. One is FocusFindings: hardenRound in burlerReviewEntry, forwarded through BurlerHalfDeps and BurlerReviewProducer.roundProfile into BuildRoundContext. The other is the hardener run-directory match in internal/burlercli/validate.go checkRoundInRunDirectory. The only tests of the flag inject it straight into CheckReviewRound or BuildRoundContext. Setting either site to false leaves every test green while every hardener focus finding outside the changed files is then refused."
    fix: "Extend the existing BurlerReview entry test in internal/shedrecipe/entries_burlerhalves_test.go (TestBurlerReviewEntry_ProfileMapping or the test that builds a round-2 context) with a harden_round true and a harden_round false case. Each case checks that the built producer's round context admits a new BLOCKING decision finding with origin focus outside the changed files only when harden_round is true. Extend TestValidateReviewVerb_RoundChecks in internal/burlercli/validate_integration_test.go with a review in the hardener run directory (the Harden-Bouncer run_subdir under the loom reviews dir) and one in the code run directory. Each carries a complete prior list and a new BLOCKING decision finding with origin focus outside the changes. Expect acceptance in the hardener directory and a refusal naming does not list in the code directory."
  - id: F12
    severity: BLOCKING
    class: design
    origin: assist:assist
    location: internal/shedadapters/bouncerdecision.go:151
    summary: "escalateUndecided's doc comment says the budget escalation's detail names the decision-class findings. The row of TestBouncer_AutoAccept that reaches it, BlockingEscalates hands a decision finding over an open BLOCKING entry to the parent as a budget escalation, asserts only the detail fragment BLOCKING findings open (hard). So the mutation report's survivors at bouncerdecision.go:151 (the id assignment) and :153 (the appended decision detail) pass every test."
    fix: "In internal/shedadapters/bouncer_escalation_test.go, set that row's wantDetail to a fragment that names the decision finding as well, such as the review also holds decision findings (D1), or add a second detail assertion for it beside the existing one. Keep the cause and snapshot assertions."
prior:
  - id: F1
    status: resolved
    answer: escalateUndecided now hands a fresh CONTINUE or CIRCLING round in the soft-cap window whose ledger holds an open non-decision BLOCKING entry to the parent with cause budget, after writing the snapshot. A CONVERGED verdict keeps cause decision, and refusalLeavingOut still refuses an accept over the open entry. The new TestBouncer_AutoAccept rows pin both. The below-window case is new and raised as F13.
  - id: F2
    status: open
    answer: The reinterpretation is now registered and the summary names the meaning change. It misses every hub that takes code_auto_accept true from the template, and the door is still two-way, so F2 is carried.
  - id: F3
    status: resolved
    answer: burler-cluster-fan.md, burler-assist-cluster-fan.md and burler-assist-consolidation.md keep origin focus on a decision-class finding that records an undriven or WARN focus entry, and burler-review-format.md says consolidation keeps it.
  - id: F4
    status: resolved
    answer: newFindings admits the origin focus exemption only when RoundContext.FocusFindings is set. TestCheckReviewRound refuses it outside a hardener round and for another class. The untested wiring of the flag is F11.
  - id: F5
    status: resolved
    answer: burler-step-review.md and burler-review-format.md both exempt a new origin focus decision finding in a hardener round from the changed-hunk and changed-file rules.
  - id: F6
    status: resolved
    answer: The below-budget row's ledger keeps round 1's alpha, and wantJudgeKept fails the row if a rejudge archived the judge outputs.
  - id: F7
    status: resolved
    answer: Accepted the fixed-differently route. A Call parses the ledger before settlement, so TestBouncer_BlockingEscalatesAnUnreadableLedger drives settleUndecided directly and asserts Awaiting, cause budget and the could-not-be-read detail.
  - id: F8
    status: resolved
    answer: The new hard-cap row with an unspent budget asserts cause EscalationBudget.
  - id: F9
    status: resolved
    answer: docs/overview.md and the field comment in internal/loomengine/config.go now name the blocking_escalates carve-out.
  - id: F10
    status: resolved
    answer: TestDecisionSwitches_SetOnTheRowsThatAskTheParent carries the testtiming keep marker.
---

### [BLOCKING] An arbiter accept on a decision escalation passes open non-decision BLOCKING findings

**Location:** `internal/shedadapters/bouncerdecision.go#escalateUndecided`

**Issue:** `escalateUndecided` routes to cause `budget` only inside the soft-cap window.
Below it, a round holding a decision finding escalates with cause `decision`, which `arbiterstop` lets the arbiter settle.
On the arbiter's `accept`, `settleDecisionAccept` calls `decisionRefusal`.
For a CONTINUE or CIRCLING verdict, `refusalLeavingOut` checks only dropped keys, so `acceptRound` settles the round even though its ledger still holds an open BLOCKING scope or design entry.
The escalation brief stops the arbiter only over a design-wrong entry, so a BLOCKING entry tagged mechanical passes.
The Code-Review rows, and the Harden-Review rows at any budget, now take this route, which this diff opened by setting `escalate_decisions` on them.

### [BLOCKING] The code_auto_accept reinterpretation misses default hubs, and the door is still two-way

**Location:** `internal/loomengine/reinterpret.go#FlagCodeAutoAccept`

**Issue:** `FlagCodeAutoAccept` returns no flag when `code_auto_accept` is absent, and the walk never runs it for an absent `loom.yaml`, since `RunsWithoutFile` is unset.
A sparse hub base holds the key only when the hub tuned it away from the template's `true`.
So the hubs whose effective value is `true`, the ones the meaning change hits, are mostly not flagged.
The orch precedent, `FlagRemoteControlDefault`, flags the absent key for this reason.
`_lym/landing/summary.md` still reads `Door: two-way` with blast radius `shed`.

### [BLOCKING] The hardener focus permission's wiring is untested

**Location:** `internal/shedrecipe/entries_burler.go:183`

**Issue:** `grep FocusFindings` over the test files finds only `roundcheck_test.go` and `roundcontext_test.go`, which set the flag by hand.
Nothing asserts that a `harden_round` BurlerReview row, or `lym burler validate` in the hardener run directory, sets it.
Nothing asserts that other rows leave it unset.

### [BLOCKING] The budget escalation's decision-finding detail is unasserted

**Location:** `internal/shedadapters/bouncerdecision.go:151`

**Issue:** The mutation report lists lines 151 and 153 as survivors.
The covering row asserts only the BLOCKING-keys part of the detail, so dropping the decision finding ids from the parent's brief goes unnoticed.

## Rejected

- The assistant's F2 at MEDIUM, carrying only the door: merged into the carried F2 at BLOCKING, since the reinterpretation itself misses hubs on the default value.
