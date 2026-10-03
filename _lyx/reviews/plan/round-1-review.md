---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/plan/10-circling-verbs.md
    summary: the circling verbs add loom refusals but no card adds their rows to contracts/specs/refusal-spec.md, which CONSTRAINTS.md's Refusal Way-Forward Invariant requires in the same commit
  - id: F2
    severity: MEDIUM
    location: _lyx/plan/06-judge-reads-facts.md
    summary: card 6's decision-rule bullet drops D6's two no-progress signals for CIRCLING, and no card says how the judge ties a recurring ledger key, which carries no class, to the gating class
  - id: F3
    severity: MEDIUM
    location: _lyx/plan/04-verdict-vocabulary.md
    summary: card 4 never says settle stops parsing strictly, so a legacy BLOCKING replay degrades in settle to an empty-pointer Stuck, and the planned replay test passes on that degraded Stuck
  - id: F4
    severity: LOW
    location: _lyx/plan/04-verdict-vocabulary.md
    summary: the legacy-harvest path states that the round is re-judged but not the route (degrade, then the burler round producer's unjudged hand-back) or its cost of one bounce unit on each row
  - id: F5
    severity: LOW
    location: _lyx/plan/08-bouncer-circling-resolution.md
    summary: card 8 calls a settled accept reaching settle unreachable, but the entry-time attach branch calls settle before the Call-entry clear runs
  - id: F6
    severity: LOW
    location: _lyx/plan/08-bouncer-circling-resolution.md
    summary: the Awaiting Reason gives the verbs a slug but names lyx loom start and lyx loom goto without saying they run in the task worktree, and under batten nothing resumes the child on a circling decision
  - id: F7
    severity: LOW
    location: contracts/specs/loom-status-spec.md:43
    summary: the status spec still defines awaiting as the PR-review hand-off at PR-Gate, and no card updates it for the circling halt at a Bouncer row
  - id: F8
    severity: LOW
    location: _lyx/plan/08-bouncer-circling-resolution.md
    summary: card 8 leaves unstated that re-entering a segment after a goto to an earlier row, with a CIRCLING round still undecided, halts Awaiting again on the stale round
  - id: F9
    severity: NIT
    location: _lyx/plan/08-bouncer-circling-resolution.md
    summary: the budget sentence's condition reads count >= budget without the ok flag that card 9's closure reports as false for a row outside a segment
---

### [MEDIUM] Circling-verb refusals have no refusal-spec rows

**Location:** `_lyx/plan/10-circling-verbs.md`; `contracts/specs/refusal-spec.md`; `CONSTRAINTS.md` § Refusal Way-Forward Invariant.

**Issue:** `CONSTRAINTS.md` binds loom to `contracts/specs/refusal-spec.md`: "Every refusal reachable through a bound module's verbs … has a row in its module's section" and "A new refusal in a bound module lands with its row and its reaching test in the same commit."
Card 10 adds new loom refusals: no status file, a run not awaiting at a Bouncer row, a latest verdict that is not `CIRCLING`, a decision already recorded, and the shared resolver's slug-required-from-the-prime and unknown-slug refusals under the `circling` group name.
Card 10's tests reach these refusals, but no card edits `refusal-spec.md`, which already carries `approve: not at PR-Gate` and the `review:` rows as precedent.
The decision record's first Constraint makes `CONSTRAINTS.md` authoritative.

**Fix:** Add `contracts/specs/refusal-spec.md` to card 10's Edit list, with one loom row per circling refusal (class, trigger and the way-forward clause each message carries), worded like the `review:` and `approve:` rows.

### [MEDIUM] Card 6's decision rule loses D6's no-progress signals

**Location:** `_lyx/plan/06-judge-reads-facts.md`, the decision-rule bullet; `_lyx/plan/05-round-facts-file.md`, recurring keys.

**Issue:** D6 defines `CIRCLING` as "the latest fresh round still carries such findings and the facts show no progress: gating counts flat or rising across the recent rounds, or gating findings on recurring or reopened keys."
Card 6 says the stencil carries "the D6 decision rule as written in the decision record", then restates it as "the facts show no progress across at least two fresh rounds".
That restatement drops both signals that define no progress, and the restatement is what the stencil author will copy.
Separately, card 5's recurring-key list comes from ledgers, whose entries carry `key`, `rounds` and `status` but no class, so the facts alone cannot show that a recurring key is gating.
The judge can tie a key to a class only through the latest review's finding that its own new ledger maps to that key.
No card says so, so the second signal has no stated mechanism.
Card 6's claims cover "the decision rule" generically, so neither signal is pinned.

**Fix:** In card 6, write the `CIRCLING` bullet with both signals as D6 gives them, and add a sentence that a recurring or reopened key counts as gating when the latest review's finding mapped to it in the ledger the judge writes is gating-class at MEDIUM or worse (or BLOCKING).
Add a claim for each signal.

### [MEDIUM] `settle` still parses strictly, so a legacy replay degrades

**Location:** `_lyx/plan/04-verdict-vocabulary.md`, the `bouncer.go` paragraph and the tests; `internal/shedadapters/bouncer.go:482`.

**Issue:** `settle` re-reads the verdict file and parses it with `parseVerdict` (bouncer.go:482), degrading on a parse error.
Card 4 makes `parseVerdict` reject legacy words and moves only `recordedVerdict` to `parseRecordedVerdict`.
On the replay path (`judged(n)` true through the alias, then `settle(ctx, n, false)`), a legacy `BLOCKING` now fails `settle`'s strict parse and degrades with "failed to parse in settle despite judged": a Stuck with an empty pointer.
D1 requires the replay to read a legacy `BLOCKING` as `CONTINUE`, which is Stuck with the ledger pointer.
Card 4's test "a legacy `BLOCKING` replays as Stuck to the round producer" passes on the degraded Stuck too, because a degrade is also a Stuck routed to the round producer.

**Fix:** State in card 4 that `settle` reads the verdict through `parseRecordedVerdict`, since both harvest sites already apply the strict `harvestedVerdict` check before they call it.
The legacy-`BLOCKING` replay test asserts the ledger pointer.

### [LOW] The legacy-harvest path does not state its route to a re-judge or its cost

**Location:** `_lyx/plan/04-verdict-vocabulary.md`, the harvest-sites paragraph.

**Issue:** Card 4 archives the judge outputs and degrades "so `judged(n)` reads false and the round is re-judged".
A degrade returns Stuck to the round producer, whose unjudged-round check (`internal/shedadapters/burler.go:302`) hands straight back to the Bouncer, which then re-judges.
That route spends one Bouncer bounce unit and one Burler bounce unit, and on the segment's last budgeted unit the degrade blocks on budget instead of re-judging.
D1's bound holds either way, since the legacy word never settles, but the plan leaves the route and its cost unstated, so the doc.go bound sentence card 4 asks for has nothing to describe.

**Fix:** In card 4, say the re-judge runs through the round producer's unjudged-round hand-back, at one bounce unit on each row, and that at an exhausted budget the run blocks on budget rather than settling.
Carry the same sentence into the doc.go bullet.

### [LOW] The settled-accept branch in `settle` is not unreachable

**Location:** `_lyx/plan/08-bouncer-circling-resolution.md`, "Settled accept reached in `settle`".

**Issue:** Card 8 calls the branch unreachable "since the Call-entry clear intercepts it".
In `Call`, though, the entry-time attach branch (`attached && b.judged(n)`, bouncer.go:253) calls `settle(ctx, n, true)` before the clear at bouncer.go:268, so a live judge attached on a round whose accept is already settled reaches the branch.
The degrade the card chooses is safe, but the false "unreachable" claim would land in a code comment.

**Fix:** Say the branch is reached only through the entry-time attach path and degrades there rather than settling a second time.

### [LOW] The Awaiting Reason mixes slug and in-worktree addressing; batten does not resume

**Location:** `_lyx/plan/08-bouncer-circling-resolution.md`, the no-decision bullet; `_lyx/plan/10-circling-verbs.md`, the docs paragraph.

**Issue:** The Reason gives `lyx loom circling accept|continue <slug>`, a form that also works from the prime, then names `lyx loom start` and `lyx loom goto --to <Name>`.
Both of those address the cwd worktree's own run (`goto`'s positional is a run-id, not a slug), so from the prime they act on the wrong run.
Under batten, `internal/battenshed/innerrun.go`'s `callAwaiting` treats every child `awaiting` as the PR-Gate hand-off.
It wraps the child's reason with "run `lyx loom approve` or `lyx loom reject` in the task worktree", and its `ReadDecision` reads only approval and rejection records, so a recorded circling decision never resumes the child.
`lyx loom start` in the task worktree is then the only resume, but neither the Reason nor the overview sentence says where to run it.

**Fix:** Have the Reason say that `lyx loom start` and `lyx loom goto` run in the task worktree.
In card 10's `docs/overview.md` sentence, note that a batten-driven run resumes from a circling decision only through `lyx loom start` in the task worktree.

### [LOW] The status spec still ties `awaiting` to PR-Gate alone

**Location:** `contracts/specs/loom-status-spec.md:43`.

**Issue:** The spec's `error` field comment reads "awaiting is the planned PR-review hand-off at PR-Gate".
After card 8, a review segment's Bouncer row also halts `awaiting`.
Card 10 updates the matching `docs/overview.md` paragraph but not this durable contract doc.

**Fix:** Add `contracts/specs/loom-status-spec.md` to card 10's Edit list and widen the comment to name the circling halt at a Bouncer row.

### [LOW] Re-entry after a goto over an undecided circling round is unstated

**Location:** `_lyx/plan/08-bouncer-circling-resolution.md`, the Call-entry clear paragraph.

**Issue:** Card 8 clears at entry on `CONVERGED` or a settled accept only.
D7's rationale names a full rewrite (`lyx loom goto` to an earlier row) as the other way out.
After such a goto, the rewritten artifact re-enters the segment, the replay finds round N still `CIRCLING` with no decision, and the run halts `awaiting` on the stale round again.
A `continue` then reviews the rewrite, so the run is not trapped.
The plan does not say this, and an operator following D7's rationale meets an unexplained second halt.

**Fix:** State the behaviour in card 8's `Call` doc-comment sentence: re-entry over an undecided circling round halts again, and `continue` then reviews the rewritten artifact.

### [NIT] The budget sentence ignores the `ok` flag

**Location:** `_lyx/plan/08-bouncer-circling-resolution.md`, the no-decision bullet.

**Issue:** `Bounces` returns `(count, budget int, ok bool, err error)`, and card 9's closure reports `ok` false for a row outside a segment, with zero count and budget.
Card 8's condition reads "When `Bounces` reports `count >= budget`", which holds for `0 >= 0` when `ok` is false.

**Fix:** Write the condition as `ok && count >= budget`, and say that `ok` false omits the sentence, as a nil seam does.

## Focus verdicts

- Cards 4 and 8 state machine: covered, apart from F3 (`settle`'s parser) and F5 (the attach path reaching `settle`).
  The entry-time attach branch applies card 4's legacy refusal and reaches card 8's decision handling through `settle`.
  Every new archive branch follows a probe or a finished run: the legacy archive follows a returned or attached run, and the settled-accept clear follows the entry probe.
- Card 4's legacy-harvest route and budget: F4.
- Cards 5 and 6 against D6's recurring-key criterion: F2.
- Card 9's lock: not a defect.
  `shedengine`'s step reads the status file through `state.ReadJSONStrict` (run.go:157), which takes and releases the status lock per call, and calls `Producer.Call` (run.go:223) outside it, so the closure cannot self-deadlock.
  A read error drops the sentence.
- Card 8 and card 10's commands: the verbs and `lyx loom start`/`lyx loom goto --to` exist, and card 8's `count >= budget` matches Shed's block rule (`episodeStuckCount >= effectiveMaxBounces`, run.go:322).
  Addressing is covered under F6, the `ok` gate under F9.
  Card 10's Intent states the attack-surface bound.
- Acceptance criteria: each maps to a card's tests.
  "The seed pass never returns Done" is already pinned by `TestBouncer_SeedCall_HappyPath`, which requires Stuck, and no card changes the seed.
  "An accept whose Commit seam fails clears and re-seeds" is card 8's test.
  `ensureFocus(round+1)` on `CIRCLING` and `BouncerRunSubdir` are implementation of D7 (the `continue` round's focus, and the "awaiting at a Bouncer row" refusal), not new scope.
- D9 and docs: both D9 lines land (cards 1 and 6), and the package docs and `docs/overview.md` sites are covered.
  `contracts/specs/refusal-spec.md` (F1) and `contracts/specs/loom-status-spec.md` (F7) are not.
