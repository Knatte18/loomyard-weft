---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:25
    summary: The board section Card commands that reach the fabric repo (issues 424 and 432) has no decision, and the record lists the fabric-reference audit as Out of scope
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:62
    summary: D4 admits a lone card over budget at a late position without saying what the batchifier does or what bounds it, though the start now grows without limit with position
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:104
    summary: D9 leaves the least-squares fit undefined for one fork, one position or few runs, leaves residual spread undefined, and does not say the Merriam session comes from the transcript layout
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:70
    summary: D5 and D9 do not state what a wrong or stale position lets through and what bounds it
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:123
    summary: D11 gives a run one profile and one set of counts, but a rebaselined run carries batches of several profiles, a fresh restart drops earlier batch records, and the card count source is unnamed
  - id: F6
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:126
    summary: D11 and the Constraints cite PATTERN-sole-parsers and an unnamed owning package for Webster-Review round files, but that pattern covers no review format and the plan writer is not told the owner
  - id: F7
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:136
    summary: D12 has the rubric restate the dependency cases and cite the Plan-Gate checks, against the Constraint that template and rubric point at the spec rule, and its delete case names the wrong card
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:138
    summary: D12 rationale says the batchifier's contiguous segments enforce card order, but the batchifier only preserves order and CheckBatchOrder in websterengine refuses a backward dependency
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:89
    summary: D7 and the Auto-mode assumptions say eleven forks while the measurement table has twelve rows, the twelfth being the resumed session's fork
  - id: F10
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:156
    summary: Auto-mode assumptions restate D2 and D3 and refer to a support-log ledger the plan writer never reads
  - id: F11
    severity: NIT
    class: scope
    location: _lyx/discussion/support-log.md:45
    summary: The Question ledger's sed environment note is neither a question nor a broken tool and belongs in no Discussion file
---

### [BLOCKING] The folded-in fabric-reference scope has no decision

**Location:** `_lyx/discussion/decision-record.md:25` (Out), `:150` (Constraints), `_lyx/discussion/support-log.md:8`.
**Issue:** `ly:orch` folded the board note `plan-fabric-reference` (issues #424 and #432) into the `batcher-v2` entry as "Card commands that reach the fabric repo".
The record still lists "The fabric-reference audit (`plan-fabric-reference` owns it)" as Out of scope, and the support log treats that note as a separate entry.
The section's three wants have no Decision, no Acceptance criterion and no stated bound:

- the plan validator refuses a card command that names a path outside the tracked tree, the weft repo above all, so Plan-Review never passes one;
- `accept-audit --batch` accepts a pathless `fabric-reference` finding on a batch that has commits when the finding comes from read-only commands, or the audit tells a read from a write in the first place;
- the plan stencil steers a corpus or fixture card to committed test data, never to live files of other worktrees.

The first two add or loosen a guard, so the attack-surface question applies: what each can now let through and what bounds it.
Read against the code, `fabricengine.RefScanner` is the single matcher behind the audit, `AcceptBatchFabricReference` requires HEAD to equal the batch's start commit, and `refusal-spec.md` rows 77 and 78 name that route, so the decisions have to say how they change.
**Fix:** add decisions for the Plan-Gate check, for the `accept-audit --batch` widening with its read-only bound, and for the template steer.
Move the audit out of Out, say which issue wants stay out, and carry the new rules into Constraints, Acceptance criteria, Open risks and the support log.

### [MEDIUM] D4 leaves the over-budget lone card unbounded

**Location:** `_lyx/discussion/decision-record.md:62`.
**Issue:** "A one-card segment is always feasible" stays coherent as worded, because the program admits a one-card segment whatever its peak (`internal/batcher/cost.go`).
What D4 omits is that the start now grows by `batch_growth` per batch with no cap, so at a late position a lone card's peak exceeds `budget` and still runs.
That bypasses the budget guard more often the longer the plan is, and the record states neither the behavior nor a bound; Open risks only says late growth is extrapolated.
**Fix:** state in D4 that such a card runs alone over budget, that its recorded `Estimate` and position show it, and that `budget` bounds only segments of two or more cards.
Add the case to Open risks and an Acceptance criterion that a one-card segment over budget at a late position is still emitted alone.

### [MEDIUM] D9's fit is undefined on thin data

**Location:** `_lyx/discussion/decision-record.md:104`.
**Issue:** The least-squares fit of start over position − 1 has no slope with one fork or with every fork at position 1, and a short sample can yield a negative coefficient that the loader would reject.
"Residual spread" has no definition.
The decision also leaves the source of "Merriam session" implicit: the tool reads Claude transcripts, where each webster-role session file owns its `subagents/*.jsonl`, and not the weft records.
The position counts every subagent of a webster session, including unattributed ones.
**Fix:** state that the fit is reported only with at least two distinct positions and otherwise marked not fitted with a reason; that a negative fitted value is shown as fitted and flagged unusable; define the spread as the residual's 25th-to-75th percentile ratio or range; and say the session and position come from the transcript layout, counting only forks that name cards.

### [LOW] A wrong position's consequence is unstated

**Location:** `_lyx/discussion/decision-record.md:70`.
**Issue:** The position comes from Go (`keptCount` in the rebaseline, 0 elsewhere), so it is not caller input.
Even so, the decision does not say what a position that differs from Merriam's real context lets through: too high after a resumed Merriam is conservative, too low after unmodelled work makes a batch exceed `budget` with nothing re-checking it.
**Fix:** add one sentence to D5 giving both directions and naming the `budget` headroom as the only bound.

### [MEDIUM] D11 assumes one profile and one set of counts per run

**Location:** `_lyx/discussion/decision-record.md:123`.
**Issue:** A rebaseline keeps the begun batches with their recorded profile and batches the tail with the then-active profile, so a run's partition can name several profiles.
`run --fresh` archives the state, so the last `state.json` omits the failed batches and recoveries of an earlier attempt while the transcripts' weight covers the whole run.
"The run's card count" does not say whether it is the plan commit's or the final partition's.
**Fix:** attribute each batch to the profile its `PartitionBatch` records; take failed batches and recoveries per batch from the last state and mark a run whose earlier attempt was archived; count cards from the final partition; and attribute a run's review findings to the profile of the majority of its cards, or mark mixed runs, so that no run is split silently.

### [MEDIUM] The owner of the review-file format is not named

**Location:** `_lyx/discussion/decision-record.md:126` and `:144`.
**Issue:** `PATTERN-sole-parsers` lists the plan, discussion, summary and recipe formats and no review format, so "where it applies" never applies and the Constraint cites a pattern that does not bind this read.
Round files are parsed by `burlerengine.ParseReview`, and a PR rework moves earlier generations under `prior-generation/reviews/`.
The plan writer is given neither fact.
**Fix:** name `internal/burlerengine`'s `ParseReview` as the reader of the round files, drop the `PATTERN-sole-parsers` citation for review files, and say which generations' round files count.

### [MEDIUM] D12's rubric clause paraphrases the rule and inverts the delete case

**Location:** `_lyx/discussion/decision-record.md:136`.
**Issue:** The Constraints say the template and rubric "point at the spec's rule, not a paraphrase of it" (`PATTERN-producer-pointer-rule`), but D12 has the rubric list the cases and point at the Plan-Gate checks.
The parenthetical "references what an earlier card deletes" names the later card, whereas `delete-before-reference` reports the deleting card that runs before a card still referencing its target.
**Fix:** have the rubric name the finding as a card-order violation of the spec's rule and point at the spec section; drop the case list.

### [LOW] D12 credits the wrong component with enforcement

**Location:** `_lyx/discussion/decision-record.md:138`.
**Issue:** `internal/batcher` keeps batches in card order; the refusal of a backward dependency is `CheckBatchOrder` in `internal/websterengine/sequence.go`.
**Fix:** reword the rationale to Plan-Gate's two checks plus webster's `CheckBatchOrder` over batches that stay in card order.

### [LOW] Fork count disagrees with the table

**Location:** `_lyx/discussion/decision-record.md:89` and `:159`.
**Issue:** The table has twelve rows, eleven first-session forks and one resumed fork, and D7 says the start of each was measured.
**Fix:** say eleven first-session forks plus one resumed-session fork, or twelve forks.

### [LOW] Auto-mode assumptions restate decisions

**Location:** `_lyx/discussion/decision-record.md:156`.
**Issue:** The D2 and D3 bullets repeat what D2, D3 and the support log's Rejected alternatives already hold, so they carry no requirement the plan writer needs, and the first bullet points at a ledger the plan writer never reads.
**Fix:** delete the D2 and D3 bullets and reword the first so it stands alone.

### [NIT] The sed note belongs nowhere

**Location:** `_lyx/discussion/support-log.md:45`.
**Issue:** The environment note records a read-only `sed -n` use against the user's global rule; it is not a question and not a broken tool, and no plan writer needs it.
**Fix:** delete the line.
