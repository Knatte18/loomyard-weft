# Fixer report — discussion round 1

Every finding's premise held against the code: `internal/loomcli/decision.go:307` (F1), the hub's `gate.yaml` `-trimpath` and `trimpathLine` (F2), `recordbatch.go:585` and the hub's `batcher.yaml` (F3), `arbiterStopEnvelope` (F4), `MaxSpawns` (F5), the recipe's plan-conflict clause (F7), `TargetGroup.Resigns` and `csharpResignTargets` (F10), `refsIntersect` (F11), the `mergeBase != nil` guard in `verifygate.go` (F12), and `VerifyFailure.Package` (F15).

## Changes

- F1 (fixed as proposed): D7 restores the decision record to its pre-append bytes and removes the amendment record when the append or commit fails. With the flag, the commit-failure way forward is to re-run the verb. The criterion now says the record holds no new entry and names that way forward.
- F2 (fixed as proposed): D9 step 1 places a path prefixed by the module path by stripping that prefix, and cites `gate.yaml` and `trimpathLine`. Turn 5 of the support log is corrected. A criterion is added for a panic frame under `-trimpath`.
- F3 (fixed as proposed): D9 step 3, the `cards` field and `PublishFailureNote` name the batch's commit range, from its recorded `StartSHA` to its recorded head. The criterion is reworded to match.
- F4 (fixed as proposed): D8 names the carrier, a new `amendment_target` arbiter prompt input that loom's wiring fills from a new top-level recipe key, which `loom-recipe.yaml` sets. It also states that the generic step envelope is unchanged. The stencil sentence, the criterion, the Scope line and Q13 are reworded to match.
- F5 (fixed as proposed): D8 states the route's bounds. The arbiter's repeats are capped by `arbiter_max_spawns`. A scope widening is bounded only by the hand-on rule, because the arbiter's entry joins the answer key. The parent's and the orch's repeats were later reworded as uncapped; see Self-check.
- F6 (fixed as proposed): D7 compares the judged round's start time with the record's latest write time. The criterion now covers an amendment appended while a round runs.
- F7 (fixed differently): I did not apply the proposed "only cards whose batch has not begun". It locks the card of the failed batch that stopped on the defect, which is D8's main arbiter case, so a re-entered Webster retries the unchanged card. Both clauses now use what auto-rebaseline accepts: a pending card or a card of a failed or in-flight batch, plus inserts after the last begun batch. D7 rewords the existing plan-conflict clause to that same rule, and a criterion is added.
- F8 (fixed as proposed): D2's bound names the second escape. The guards for both escapes are the card gate's `lyx loom lint-pattern` and the verify gate's headroom lint.
- F9 (fixed as proposed): D8 fills the target only once the current generation's plan is approved. The rationale and the criterion are reworded to match.
- F10 (fixed as proposed): in D3, a re-sign arrow under `Extends` is a blocking finding whose way forward is to move the ref under `Edit`. The criterion is added.
- F11 (fixed as proposed): D9 step 3 states the overlap rule as D9's own.
- F12 (fixed as proposed): D1's rationale limits the claim to a run with a recorded parent and states what holds when no parent is recorded.
- F13 (fixed by the finding's second option): D1 makes a `removes` entry whose ID a card also fences a blocking plan-gate finding that names the bullet and the card. The criterion is reworded. I first applied the ordering option, then switched; see Self-check.
- F14 (fixed as proposed): D1's way forward names `--amends-plan` once the plan is approved.
- F15 (fixed as proposed): in D9 step 1, an opaque identity's bare or `./`-relative locations go into `untraced`, and other unplaced locations are still dropped. The `untraced` definition, the Open risk and the criterion are updated.
- F16 (fixed as proposed): criteria are added for the rubric and the plan-writer stencil (D6), the Plan-Burler-Review instruction (D7) and the orch stencil (D8).
- F17 (fixed as proposed): D5's estimate clause moved into its rationale. The second Open risk is deleted, and Turn 6's line that credited it is corrected.

## Self-check

- fix-claims, MEDIUM: D8's "capped by the spent bounce budget" is not a cap for the orch or the parent. Fixed differently from the fork: D8 now states that parent repeats have no cap and pause on each spent budget for the parent's resume. This is merged with fix-proposal's LOW finding 4.
- fix-consistency, LOW: the Scope docs bullet omits the recipe spec. Fixed: `shed-recipe-spec.md` is added.
- fix-consistency, LOW: support-log Turn 6 still says the stop envelope. Fixed.
- fix-consistency, LOW: support-log Turn 6 credits the deleted Open risk. Fixed.
- fix-way-forward, MEDIUM: an amendment kept by the start-time removal rule can no longer pass the goto guard, which compared against the settle time. Fixed: the guard's admission and D7's bound compare the record's write time with the start of the round that the settle judged. The criterion is updated.
- fix-way-forward, LOW: the commit-failure way forward did not say it applies only with `--amends-plan`. Fixed: a call without the flag keeps `lyx fabric commit`.
- fix-proposal, MEDIUM: F7 as proposed locks the failed batch's card. Fixed differently, as recorded under F7.
- fix-proposal, MEDIUM: F13's ordering option let a contradictory plan pass. Fixed by switching to the finding's blocking-finding option.
- fix-proposal, MEDIUM: my widening of F15 sent standard-library and dependency frames into `untraced`. Fixed: other unplaced locations are dropped again, and only an opaque identity's locations go to `untraced`.
- fix-proposal, LOW: "approved" must mean the current generation's plan. Fixed in D7's refusal and D8.
- fix-proposal, LOW: F5's parent cap. Merged with fix-claims above.

## Disputed

None.

## Deferred

None.
