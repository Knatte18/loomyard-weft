# Review convergence: the judge decides converged, continue or circling from fresh reviews

## Goal

A review segment stops when it has converged, judged from fresh reviews, and escalates to the operator when it circles.
The Bouncer judge stops acting as a second reviewer of the artifacts.
It becomes a narrow sit-rep agent that reads Go-computed facts about the rounds and decides one of three outcomes: converged, continue or circling.
Review findings carry a finding class, so the facts show which kind of finding keeps recurring, and the judge stops the loop on gating-class findings rather than on every finding.

## Scope

In:

- The Bouncer judge pass (`internal/shedadapters`, `contracts/stencils/bouncer/bouncer-template-judge.md`): new verdict vocabulary, new reading list, new outcome mapping.
- A Go-written per-round facts file the judge reads instead of the artifacts.
- A finding-class field on burler's review contract (`internal/burlerengine`, `contracts/stencils/burler/burler-step-2-review.md`) and a class mapping in each of the three loom review rubrics.
- The circling escalation: an Awaiting halt and two operator verbs that resolve it.
- A `judge` model key in loom's config, taken by every Bouncer row.
- All three review segments: Discussion-, Plan- and Webster-Review.
- Docs and invariants: `CONSTRAINTS.md`'s Review Round Invariant, the `internal/shedadapters` and `internal/burlerengine` package docs, `docs/overview.md` where it describes the Bouncer or the review contract, removal of the completed roadmap entry and the draft `manifest/designs/review-finding-classification.md`.

Out:

- `internal/treadleengine` and its own judge; it is not a loom review segment.
- The burler fixer's behavior: it still fixes every finding, every severity.
- The Bouncer seed pass's job (setting round 1's focus from the artifacts) and the `Skip` seam; only the seed's model changes.
- Go-side enforcement of the judge's ledger carry-forward rule.
- Model choice for the reviewers (`review` key) and the short-form sweep of other model keys; both belong to `pattern-init` and `convergence-plan`.

## Decisions

### D1. The judge's verdict is one of `CONVERGED`, `CONTINUE`, `CIRCLING`

Decision: the judge's verdict file carries `verdict:` exactly `CONVERGED`, `CONTINUE` or `CIRCLING`, with the existing quoted one-line `rationale`.
`CONVERGED` maps to the old approved branch (Approve seam, Commit seam, Done, ledger pointer).
`CONTINUE` maps to the old blocking branch (Stuck to the round producer, ledger pointer).
`CIRCLING` maps to a new Awaiting halt (D7).
Everywhere the Bouncer treats an on-disk APPROVED verdict as "settled earlier" (the clear-and-re-seed at Call entry), it treats `CONVERGED` the same way.
On read only, a legacy `APPROVED` parses as `CONVERGED` and a legacy `BLOCKING` as `CONTINUE`; the judge prompt asks only for the new words.
Rationale: three outcomes need three words; reusing APPROVED/BLOCKING would keep the "a reviewer's approval" reading that made the judge act as a reviewer.
The read-only aliases keep a run dir written by the old binary from being re-judged as unsettled, which would let a rework generation pass on a stale round instead of clearing and re-seeding.

### D2. The judge reads facts and the latest review, never the artifacts, the fixer report or code

Decision: the judge prompt's reading list is, in order: the round's facts file (D3), the latest round's review file, the previous ledger.
The artifact-path list leaves the judge prompt entirely, and so does any instruction to check fixes against the artifacts.
The fixer report is not on the list.
The rubric stays embedded, scoped in the prompt to writing the focus file and to reading what the rubric's classes mean; it is not a mandate to re-review.
The prompt states: fixed findings never count as convergence, since a fix is new text no fresh reviewer has seen;
the review file's own top-level `verdict:` is not a convergence signal.
The judge still writes all three files: verdict, ledger and next-round focus.
Rationale: the operator's role for the judge is a quick sit-rep, and the rework-generations run showed a judge that reads artifacts and fixes declares "all fixed" converged with no fresh review behind it.
The review file stays on the list because the ledger maps findings to keys and the focus file names where to look; both need the findings' locations and summaries, not the artifacts.

### D3. Go writes a per-round facts file before each judge spawn

Decision: before spawning the judge for round N, the Bouncer writes `round-<N>-facts.md` in the run directory, deterministically regenerated on every judge call for that round.
It carries, for every round 1..N:

- finding count, count per severity and the highest severity;
- count per class, and the gating-class count at MEDIUM-or-worse (D5);
- the trend: those counts side by side across rounds, so a reader sees rise, fall or plateau without arithmetic.

From the ledgers of rounds 1..N-1 it also lists every recurring key: a key whose `rounds` holds two or more rounds, with those rounds, and marked reopened when an earlier ledger had it `resolved` and a later one `open`.
The Bouncer reads review files through burlerengine's own review parser, not a second one.
A round whose review no longer parses (an old-format review, a hand-edited file) gets a row naming the parse error instead of counts; it never fails the judge call.
The reviewer's top-level `verdict:` is not in the facts.
The facts file is part of the committed run directory, like the ledger, and is the "round record" that carries per-class counts.
Rationale: the facts must be durable and human-readable, so the same class recurring across rounds is visible after the run, and a file keeps the judge's prompt-through-file contract.
Fail-soft rows keep an in-flight segment that straddles the binary update from halting on history it cannot change.

### D4. Every review finding carries a `class`

Decision: burler's review contract gains a required per-finding `class`, exactly one of `design`, `scope`, `decision`, `consistency`, validated fail-loud in burlerengine's review parser like `severity` (missing, empty or unknown is a parse error).
Generic meanings, stated once in burler's review step stencil:

- `design`: the approach or behavior is wrong: a wrong or missing design decision, a false premise, a correctness bug.
- `decision`: something the next stage needs is left undecided or ambiguous.
- `scope`: an enumeration or coverage gap (missed call sites, files, cases) that a compiler or mechanical sweep finds better; a reviewer seeing one recur raises it once as a `design` finding about the method, not as N missing sites.
- `consistency`: the text disagrees with itself or with its sources: names, cross-references, stale wording.

Each of the three loom rubrics (`loom-rubric-discussion-review.md`, `loom-rubric-plan-review.md`, `loom-rubric-webster-review.md`) states what `design` means in its own catchment (Discussion: is the design right; Plan: batching, sequencing, sizing and `verify:` correctness; Webster: the implementation matches the plan and the invariants and is correct).
Rationale: severities alone are the reviewer's noisy labels; the class shows whether a recurrence is a design problem or an enumeration the toolchain owns, which is the signal the judge needs to tell circling from progress.

### D5. `design` is the one gating class, in every segment

Decision: `design` is the gating class for all three segments, defined once as a burlerengine constant, not per rubric.
Class governs who decides and when the loop stops, never whether a finding is fixed: the fixer fixes every finding of every class and severity, exactly as today.
Rationale: one gating class keeps the rule explainable and testable; each rubric's own definition of `design` already narrows it to that segment's catchment.

### D6. The judge's decision rule

Decision: the judge prompt states the rule the judge applies, with the judge free to depart from it only by naming the finding and why in its rationale:

- `CONTINUE` while the latest fresh round carries a gating-class finding at MEDIUM or worse, or a BLOCKING finding of any class.
- `CONVERGED` when the latest fresh round carries neither, whatever the earlier rounds or the fixer did.
- `CIRCLING` when the latest fresh round still carries such findings and the facts show no progress: gating counts flat or rising across the recent rounds, or gating findings on recurring or reopened keys.
  Circling needs evidence from at least two fresh rounds.
- A departure is allowed only for a finding whose own summary shows it is mislabeled (a nitpicky MEDIUM, a `consistency` problem filed as `design`); the rationale names the finding ID.

Only two rules stay mechanical in Go: a review segment's Done is reachable only from a judge verdict over a round whose review exists (the seed pass never returns Done, and the `Skip` seam is unchanged), and `max_bounces` stays the upper bound.
Rationale: the operator rejected a pure mechanical rule (severities are noisy, a reworded old finding looks new), so the judge decides; writing the default rule into the prompt keeps its decisions consistent, and requiring a named finding for a departure keeps them auditable.

### D7. Circling halts as Awaiting; the operator resolves it with `lyx loom circling accept|continue`

Decision: a `CIRCLING` verdict returns Awaiting, with the ledger as the pointer and a Reason naming both verbs and the slug.
Re-calling the Bouncer while round N's verdict is `CIRCLING` and no decision is recorded returns Awaiting again and spawns nothing.
Two new verbs, `lyx loom circling accept [<slug>]` and `lyx loom circling continue [<slug>]`, record the operator's decision in the run directory for round N (slug addressing as in `lyx loom review`: a task worktree addresses itself, the prime needs the slug).
On the next Call:

- `continue` returns Stuck to the round producer, exactly as `CONTINUE` does; it counts against the segment's `max_bounces`, so a circling at the last budgeted round blocks on budget, and the existing `lyx loom goto` gives a fresh budget.
- `accept` settles the segment exactly as `CONVERGED` does (Approve seam, Commit seam, Done), and records the acceptance so a later re-entry of the segment clears and re-seeds as it does after a converged generation, never settling again on the same accept.

Bound: the verbs refuse unless the addressed run is awaiting at a Bouncer row whose latest judged round's verdict is `CIRCLING`, and refuse a second decision for the same round.
`accept` lets exactly one circling round pass without a converged judgment; it skips no seam, and the decision file in the committed run directory records the decision for that round.
The verbs check no caller identity, like `lyx loom review`; the trust boundary is the operator's shell, the same as for every other loom verb.
Rationale: Awaiting is Shed's planned human hand-off and spends no budget; `lyx loom goto` from an awaiting row moves only strictly before it, so without a verb the only way out would be a full rewrite.
A recorded decision mirrors PR-Gate's approve/reject records: no record means Awaiting, so a stray resume never guesses.

### D8. The Bouncer runs on its own `judge` model key

Decision: loom's config gains `judge: sonnet[medium]` (short form), validated like the other model keys.
Every Bouncer row's seed and judge passes take it as their default, in place of the `review` key; a row's own `model`/`effort`/`version` keys still override.
BurlerRound rows keep the `review` key.
Rationale: sharing `opus[effort=high]` with the reviewers is part of why the judge acted as one, and the narrowed job (D2) fits a fast, cheap model (operator, 2026-10-02).
The seed shares the key because it is a Bouncer pass and its focus never caps what the opus round-1 reviewer looks at.

### D9. Invariant and docs

Decision: `CONSTRAINTS.md`'s Review Round Invariant gains two lines:
a review segment converges only on a judge verdict over a fresh review, and fixed findings never converge on their own;
a finding's class decides who decides and when the loop stops, never whether it is fixed.
The completed roadmap entry is removed, and `manifest/designs/review-finding-classification.md` is deleted, its lasting content (the classes, the gating rule, the severity-ladder trap) moving into the burlerengine package doc.
Rationale: both lines are cross-cutting rules a reviewer must enforce; the draft doc said it was not a spec, and as-built behavior belongs in package docs.

## Constraints

- `CONSTRAINTS.md` is authoritative; the Review Round Invariant's "every finding fixed, all severities" stays.
- Stencils stay conditional-free: every marker non-empty, no `{{if}}`/`{{range}}`; variants are held in Go (as `focusSchemaMarkers` does).
- The judge still writes exactly three files, and the live-agent probes (entry-time, seed, judge, re-bounce) keep their "probe before archive, attach if live" behavior for the new branches, including the circling re-call.
- The pointer rule holds: a pointer names a verified-existing file or is empty.
- The cancellation rule holds: a harvested verdict, including `CIRCLING`, is returned as its mapped outcome even after cancellation.
- Agents run in interactive tmux through shuttle; the judge spawn is unchanged in that respect.
- Docs land in the same commit; Markdown uses semantic line breaks.

## Auto-mode assumptions

- No operator answered; every decision above is an auto-pick recorded in the support log's question ledger.
- The board entry's "end the round loop when a round returns zero gating-class findings" is read as the judge's default rule (D6), not a Go rule, because the same entry says only two things stay mechanical.
- `design` alone gates, per the draft design doc, in all three segments.
- An escalation needs a resolution route the operator can take without editing files, hence the D7 verbs.
- The seed pass moves to the `judge` key with the judge.
- In-flight runs may straddle the binary update; D1's aliases and D3's fail-soft rows cover that, nothing else does.

## Open risks

- A sonnet[medium] judge may misread a trend; D6's written default and named-departure rule bound it, and `max_bounces` caps the cost.
- Reviewers may file real problems under a non-gating class to avoid blocking; the fixer still fixes them, and D6 lets the judge treat a mislabeled finding as gating, but only from its summary.
- Requiring `class` adds a new way for a review file to fail parsing; burler's existing malformed-review retry covers it, at the cost of a retried round.
- `continue` at the last budgeted round blocks immediately on budget; the Reason must say so, or the operator meets a second halt without knowing why.
- The `accept` record must stay distinguishable from a fresh accept on re-entry; getting that wrong lets a reworked artifact pass with no review.

## Acceptance criteria

- `ParseReview` rejects a finding with a missing, empty or unknown `class`, and accepts each of the four values.
- The judge verdict parser accepts exactly `CONVERGED`, `CONTINUE`, `CIRCLING`, maps legacy `APPROVED`/`BLOCKING` to `CONVERGED`/`CONTINUE`, and rejects anything else.
- The rendered judge prompt names no artifact path and no fixer report, and names the facts file, the latest review and the previous ledger.
- `round-<N>-facts.md` for a fixture of three rounds shows per-round finding count, per-severity counts, highest severity, per-class counts and the gating count, and lists a key seen in rounds 1 and 3 as recurring and, when round 2's ledger had it resolved, as reopened.
- A facts fixture whose round-1 review lacks `class` renders a parse-error row for round 1 and still lets the judge call proceed.
- The rework-generations case: round 1's review has MEDIUM `design` findings, the fixer report says all fixed; the facts file shows the gating count at round 1 as non-zero, and a judge verdict of `CONTINUE` routes to the round producer.
- `CONVERGED` runs Approve, Commit and returns Done with the ledger pointer; re-entering the segment afterwards clears and re-seeds.
- `CONTINUE` returns Stuck to the round producer with the ledger pointer.
- `CIRCLING` returns Awaiting with the ledger pointer and a Reason naming both `lyx loom circling` verbs; re-calling with no decision returns Awaiting and spawns nothing.
- After `lyx loom circling continue`, the next Call returns Stuck; after `lyx loom circling accept`, the next Call runs Approve and Commit and returns Done, and a later re-entry clears and re-seeds rather than settling again.
- Both verbs refuse a run not awaiting at a Bouncer row, a latest verdict other than `CIRCLING`, and a second decision for the same round.
- The seed pass never returns Done.
- A judge whose process is still alive when the Bouncer is re-called on a `CIRCLING` round is attached and harvested, not respawned and not archived over.
- Bouncer rows resolve their model from `judge`, BurlerRound rows from `review`; a row-level model override still wins; an invalid `judge` value fails config load.
- `CONSTRAINTS.md`'s Review Round Invariant carries both D9 lines.

## Notes for the plan writer

- The Bouncer's verdict parsing and the approved/blocking constants live in `internal/shedadapters/bouncerfiles.go`; the four-mode branch is `Bouncer.Call` in `bouncer.go`.
- Bouncer rows get their model in `internal/shedrecipe/entries_bouncer.go` from `env.ReviewModel`; `Env` and loom's config (`internal/loomengine/config.go`, `template.yaml`) need the new key.
- Shed's Awaiting re-calls the same producer on resume; PR-Gate (`internal/landingshed/prgate.go`) is the existing model for a producer that waits on a recorded human decision.
- The judge stencil's header comment and the `shedadapters` package doc both describe the old reviewer role and the APPROVED/BLOCKING mapping; both need rewriting, not patching.
