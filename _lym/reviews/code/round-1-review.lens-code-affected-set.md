```yaml
lens: code-affected-set
outcome: reported
```

I reviewed the code-affected-set lens: everything outside the diff's own lines that reads or describes what it changed.

**Per change, the set found, covered or not**

1. **`blockingSendsToRule` / `openBlocking` replaced by `blockingEscalationDetail` / `openBlockingKeys`** (`internal/shedadapters/bouncer.go`).
   - Callers: only `settleUndecided`, which the diff updates. No other caller remains (grep found none). Covered.
   - Docs: the `BouncerConfig.BlockingEscalates` doc, the Bouncer type doc, the `settleUndecided` doc and `internal/shedadapters/doc.go` are all updated. Covered.
   - Tests: `TestBouncer_AutoAccept`, `TestBouncer_SoftCapRule` and its `softCapPath` comment, and `TestBouncerEntry_BlockingEscalatesAndColorSegment` are updated. The `TestBouncer_SoftCapRule` doc ("five steps … every path that reaches the rule") is still true. Covered.
2. **What `blocking_escalates` means.**
   - Covered: `contracts/specs/shed-recipe-spec.md`, the `budget` cause bullet and "decide nothing" line in `bouncer-template-escalation.md`, the `code_auto_accept` comment in `internal/loomengine/template.yaml`, and `entries_bouncer.go` (the key was already accepted).
   - **Not covered:** `docs/overview.md:1515` and the `CodeAutoAccept` comment in `internal/loomengine/config.go:520` (finding 4).
   - **Not covered:** the config-reinterpretation rule in `PATTERN-config-layers` (finding 3).
3. **The `escalate_decisions` / `decisions_escalated` switches on the Code and Harden rows.**
   - Covered: the recipes and the testdata copy, the shed-recipe-spec sentence, the `decisionswitches_test.go` header, and the HardenFix key set (it takes BurlerFix's keys, so `decisions_escalated` is accepted).
   - Covered: the `loom-rubric-code-review.md:119` "deferred to the parent" wording still holds, since escalation now does the deferring.
   - **Not covered:** the arbiter's settle rule in `bouncer-template-escalation.md` and `bouncer-fragment-decision.md` (findings 1 and 2).
4. **New origin value `FocusOrigin` = `"focus"`** (`roundcheck.go`).
   - Covered: no parser restricts the origin values (the grep found no validator), and `burler-review-format.md`, `burlerengine/doc.go` and the `CheckReviewRound` doc are updated.
   - Gap: `FocusOrigin` sits in `roundcheck.go` while its siblings `WebsterOrigin` and `ParentOrigin` sit in `parentseat.go`. This is placement only; I raise no finding.
5. **Hardener rubric now raises focus findings.**
   - Covered: `harden-rubric-review.md`, the format exemption in `burler-review-format.md`, and the Harden-Bouncer `escalate_decisions` row.
   - **Not covered:** the arbiter's path to rule on these findings (finding 1).
6. **Commit messages.**
   - The Root-Cause trailer names the three defects the diff removes.
   - The Loosened-Check trailer names `TestBouncer_SoftCapRule` with a behavioural cause, not "flaky", and the behaviour it points to is asserted in `TestBouncer_AutoAccept`.
   - No Pattern-Departure trailer. No finding.
7. **Declared PATTERN changes:** the directive declares none, so there is nothing to land.

**Findings**

1. **[BLOCKING, design]** `contracts/stencils/bouncer/bouncer-fragment-decision.md:9`
   - **Summary:** the arbiter cannot rule first on an `origin: focus` finding, so a hardener's undriven focus entries always go to the parent.
     - The fasit's item 3 asks the arbiter to order another round or accept with a stated reason for each entry.
     - The summary claims "The arbiter rules first: it orders another round, or accepts with a reason per entry."
   - **Evidence:**
     - The fragment lets the arbiter settle only "each finding the board entry and the decision record answer".
     - `bouncer-template-escalation.md` lines 40–56 allow a settle only "when a recorded operator decision answers the envelope's question directly". Otherwise it records nothing and reports back.
     - An undriven focus entry is never answered by a recorded operator decision.
     - The diff changes neither stencil, and neither does `loom-template-arbiter.md`.
2. **[BLOCKING, design]** `internal/arbiterstop/arbiterstop.go`, the `Eligibility` decision case; the fix site could also be `bouncerdecision.go` `settleDecisionAccept`
   - **Summary:** at a spent budget, a round with a held decision finding and an open BLOCKING finding of another class escalates with cause `decision`, not `budget`. The arbiter may settle that cause, and its accept passes the open BLOCKING findings without the parent. That breaks item 2 ("budget spent with a BLOCKING finding open stops and asks its parent").
   - **Evidence:**
     - `settleUnlessRefused` runs `routeDecisions` before `settleUndecided` (`bouncer.go:1081`).
     - `settleDecisionAccept` → `decisionRefusal` → `refusalLeavingOut` checks BLOCKING entries only on a CONVERGED verdict (`bouncer.go:1122`). It then calls `acceptRound`.
     - `arbiterstop.go` `Eligibility` admits `EscalationDecision`.
     - The escalation stencil forbids the arbiter only to accept over a design-wrong finding (line 54), not over any BLOCKING one.
     - `bouncer-fragment-decision.md` says "An `accept` passes the round over every BLOCKING finding listed".
3. **[BLOCKING, design]** `internal/loomengine/template.yaml:53`
   - **Summary:** the diff changes what a hub's `code_auto_accept: true` means but ships no `configreg.Reinterpretation`. `PATTERN-config-layers` requires one for a meaning change: it rewrites the value to its old meaning, or flags it with a way forward. The old meaning was "accept … BLOCKING ones included"; the new one is "a round with an open BLOCKING finding is never accepted this way".
   - **Evidence:** `pattern/PATTERN-config-layers.md:136` and the template comment diff.
   - **Caveat:** the summary argues that the recipe key, not the config value, carries the change, and the operator ordered it on 2026-11-22. The handler should weigh whether that makes it no meaning change of the value.
4. **[LOW, consistency]** `docs/overview.md:1515`
   - **Summary:** it still says `code_auto_accept` makes the perch "accept a harvested round at a spent budget", with no carve-out for an open BLOCKING finding.
   - **Evidence:** `docs/overview.md:1515`. The same gap is in the `CodeAutoAccept` field comment at `internal/loomengine/config.go:520` ("A flag true makes the perch accept a harvested round at a spent budget instead of running the soft-cap rule").

**Tool calls**
1. Grep for leftover callers of the renamed functions, origin constants and validators, mentions of `blocking_escalates`, and `code_auto_accept` docs: the manifest held no repo-wide search for them.
2. Read the review parsers for an origin check, `docs/overview.md` around 1515, the `config.go` field comment, and grepped stencils and docs for the decision-switch wording: docs and config readers were outside the manifest.
3. Read the arbiter's settle rule in the escalation stencil, the decision fragment, and the decision wording in the judge and harden stencils: the arbiter's own rule text was not in the manifest.
4. Read the `TestBouncer_SoftCapRule` doc comment, the arbiter prompt grep, and the list of stencils that mention the arbiter: to check the test doc against the loosened check and to find the arbiter stencils.
