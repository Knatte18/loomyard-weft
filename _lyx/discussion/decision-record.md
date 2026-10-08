# Decision record — review-loop-convergence

## Goal

Review segments stop circling to their budget on fresh edge-case keys.
From round 2 on, a segment converges once its latest round carries no BLOCKING finding and no MEDIUM-or-worse `design` finding on a key already open in an earlier round, and the fresh findings it closes on are carried into the decision record: Discussion-Review's for Plan-Write and rework to check, Plan-Review's and Webster-Review's for Describe to turn into items checked by hand; round 1 keeps today's rule.
The template budget drops to three rounds, each review segment can set its own review and fix models, the fixer can be told to start only after the review is accepted, and a recorded circling decision is resumed by `lyx loom resume`, the step the hub's role already names (issue #446).

## Scope

In:

- The judge's decision rule (`decisionRuleMarker` in `internal/shedadapters/bouncerprompt.go`) and the judge stencil's convergence wording (`contracts/stencils/bouncer/bouncer-template-judge.md`).
- A Go-written carry-over of a closing round's open findings into the decision record's `## Open risks` section, on every Bouncer `Done` that settles a judged round, and the Plan-Write, rework and Describe stencils (`contracts/stencils/loom/loom-template-plan.md`, `contracts/stencils/loom/loom-template-rework.md`, `contracts/stencils/landing/landing-template-describe.md`) telling each what to do with an entry, and one sentence in the Plan-Review rubric (`contracts/stencils/loom/loom-rubric-plan-review.md`) keeping an entry out of its answer key.
- `review_max_bounces` template default from 5 to 3 in `internal/loomengine/template.yaml`, with its comment and the `review_circling_checkpoint` comment updated.
- Per-segment review and fix model keys in `loom.yaml`, falling back to `review` and `fix`.
- A `loom.yaml` option for when a round's fixer starts: beside the reviewer (default, today) or after the review is accepted.
- `lyx loom resume` over a run awaiting at a Bouncer row with a recorded circling decision; the circling verbs' `resume` output field, the escalation brief and the circling verb doc name `lyx loom resume`.
- PATTERN.md's `PATTERN-review-round` entry and `pattern/PATTERN-review-round.md`, plus the package `doc.go` files of every package whose observable behavior changes.

Out:

- Auto-accept at the round cap: dropped by the operator (relayed by `ly:orch`, 2026-10-08). A spent budget still escalates to the parent, unchanged.
- Lens fans for Discussion and Plan: task `review-lens-fan`.
- Per-segment judge models, per-segment budgets, and per-segment fixer start: nobody asked for them.
- `lyx loom approve` / `reject` outputs and PR-Gate's awaiting refusal in `loom resume`: unchanged.
- Batten's decision reading (`battenshed` `ReadDecision`): it keeps reading approvals and rejections only.

## Decisions

### D1 — Lighter convergence rule from round 2

Decision: from round 2 on, the judge rules `CONTINUE` while the latest round carries any BLOCKING finding, or a gating-class (`design`) finding at MEDIUM or worse on a key open in an earlier round, unchanged or reopened; otherwise `CONVERGED`.
Round 1 keeps today's rule: `CONTINUE` while it carries any BLOCKING finding or any gating-class finding at MEDIUM or worse.
The earlier-open set is a new Go-rendered list in the round facts: every key whose status is `open` in at least one ledger before the latest round.
Today's recurring-key list does not serve, since it needs a key open in two earlier ledgers and is empty at round 2; it stays as it is for CIRCLING.
An earlier ledger that is missing or fails to parse renders a parse-error line in the earlier-open list, naming its round, and the lighter rule states that such a line means the judge rules by round 1's rule, so a degraded read never empties the guard.
`decisionRuleMarker` renders the variant by round, as it already does for the CIRCLING bullet; the CIRCLING bullet and the checkpoint are unchanged.
The judge stencil's sentence "Fixed findings never count as convergence" is reworded to state the new bound: a fix converges only for a key first raised in the closing round, from round 2 on, and only with the carry-over of D2.
Bound: the rule lets a round-2-or-later round converge over fixes of fresh MEDIUM `design` keys no fresh reviewer has seen; it never lets through a BLOCKING finding, a MEDIUM `design` finding the judge maps to a key in the earlier-open list, a MEDIUM `design` finding in a round whose earlier ledgers did not all read, or any round-1 fix.
Rationale: the 2026-10-08 escalations closed on APPROVED reviews whose only gating findings were fresh edge-case keys, so another round would only have raised more; keeping round 1 on today's rule gives every first-round fix, which carries the bulk of the changes, one fresh review.

### D2 — Carry-over of the closing round's open findings

Decision: when a Bouncer settles a judged round as `Done` (a `CONVERGED` verdict, or a recorded circling `accept`), Go writes the segment's carry-over entry into the decision record's `## Open risks` section, listing every `status: open` entry of that round's ledger at MEDIUM or worse, plus every open entry with no class or severity, rendered as `unlabelled`.
Each listed finding carries its key, class, severity and the rounds it was open in, taken from the ledgers' own round numbers as the round facts take them, never from the judge-written `rounds` list.
The entry also names the round's review and fixer-report paths, anchor-relative (for example `_lyx/reviews/discussion/round-2-review.md`), never absolute.
The entry names its segment, its round and which of the two closed the round: under `CONVERGED` a listed finding was addressed by the round's fixer and no fresh reviewer has seen the fix; under a circling `accept` it may also be unfixed.
Entry shape: an opening line `<!-- lyx:carry-over <segment> -->`, the entry's bullets, and a closing line `<!-- /lyx:carry-over <segment> -->`; the entry is everything between the two lines, inclusive.
Each segment owns at most one entry, identified by its segment name: a settle replaces the bytes between that segment's markers, appends a new entry at the end of the section when none exists, and removes it when the ledger has no such entry.
The segment name (`Discussion-Review`, `Plan-Review`, `Webster-Review`) reaches the Bouncer as a new told config field, never through `Name`, which stays a log identity.
The entry is Go-owned: an edit a later fixer makes between the markers is overwritten by the next settle, and every byte outside them is kept.
So a later generation's `Done`, after a re-entry, a rework generation or a `goto`, supersedes the earlier generation's entry, and a replay writes the same bytes.
A skipped review (the Bouncer's skip seam settling `Done` with no ledger) leaves the segment's entry as it is: no round ran, so the last reviewed generation's entry still describes the artifact.
An entry stays until its segment's next `Done` that settles a judged round; one written by a generation that later failed to reach `Done` therefore remains, and its paths may point into an archived run directory until that next `Done` rewrites it.
The write goes through `discussionparser` (a new write path beside `AppendDecision`, which writes `## Decisions` only), so `PATTERN-sole-parsers` holds.
It runs before Approve and Commit, and on a circling accept before the accept's settle write; a failed write returns settle's error, as a failed Approve does.
A failed accept then stays pending and its re-call retries the write; a failed `CONVERGED` settle is cleared and re-seeded as today, and the new generation's `Done` writes the entry again.
The write and its commit are one Go seam on the Bouncer, wired for all three review segments, which commits the decision record alone through fabric's anchored commit; no existing seam's pathspec widens, and a failed commit returns settle's error as a failed write does.
Consumers, each told by its own stencil:
the Plan-Write and rework stencils say a carry-over entry lists findings whose fix in the record no fresh reviewer has seen, to check against the code while exploring and never to plan as scope;
the Describe stencil carries each listed Plan-Review or Webster-Review finding into the description's items to check by hand, worded as the finding and its location, never as a review round.
The Plan-Review rubric (`contracts/stencils/loom/loom-rubric-plan-review.md`) gains one sentence in its answer-key bullet: a carry-over entry in `## Open risks` is not part of the answer key, and it licenses no card's scope.
The Webster-Review rubric is unchanged, since it holds the code against the plan and never reads the record.
Marker recognition: a marker line is a line equal to `<!-- lyx:carry-over <segment> -->` or `<!-- /lyx:carry-over <segment> -->` after trailing whitespace is trimmed, counted only inside `## Open risks` (from its heading to the next H2) and outside fenced blocks.
Marker text anywhere else, inline in a sentence, in another section or in a fence, is ordinary prose.
Bound: the write touches only its segment's own entry inside `## Open risks`.
Splicing at the recognised marker lines keeps every other byte by construction, so the marker recognition and the one-line rendering below are what bound the write.
The check that can fail runs on the result before the file is replaced: the write re-reads the new bytes and requires the same H2 heading sequence as the prior file, exactly one well-formed pair for its segment (none after a removal), and every other segment's entry byte-identical to the prior file's.
Every judge-written field is rendered on one line, with line breaks and control characters replaced by spaces and backticks escaped, so a field cannot open a heading, a fence or a marker line.
A record with no `## Open risks` heading, a failed result check, or a marker set in `## Open risks` that is not zero or one well-formed pair per segment is left as it was, and settle returns an error naming the file.
A marker set is malformed when it holds an opening line with no closing one, a closing line with no opening one, two pairs for one segment, a closing line naming another segment than the open one, or a pair opened inside another.
An entry is informational: it names findings for the next producer and reviewer to check, never a decision to implement.
The section grows by at most one entry per review segment, whatever the number of generations.
Rationale: the operator's precedent of 2026-10-08 recorded the remaining calls in the decision record by hand; Go writes it from the ledger the judge already produces, so no LLM output contract changes, and `## Open risks` is an existing section the discussion gate already accepts.

### D3 — Template budget of three rounds

Decision: `review_max_bounces` in `internal/loomengine/template.yaml` goes from 5 to 3; a spent budget escalates to the parent exactly as today.
Its comment and the `review_circling_checkpoint` comment state that with both at 3 a run reaches the budget escalation at round 3 before a CIRCLING verdict could change the outcome.
`review_circling_checkpoint` stays 3.
Test-kit defaults that mirror the template follow it where a test pins the template value.
Rationale: operator decision, 2026-10-08 ("it is mostly fine after three rounds"); the prime's own `loom.yaml` already carries 3.

### D4 — Review and fix models per segment

Decision: `loom.yaml` gains six keys, `discussion_review`, `discussion_fix`, `plan_review`, `plan_fix`, `webster_review`, `webster_fix`, each a `ModelSpecList` like `review` and `fix` (one model-spec, or a per-round list whose last entry serves every later round).
An empty value (`""`, the template value) takes the run-wide `review` or `fix` list; a set value replaces it for that segment's BurlerRound row.
Every set entry is validated through `modelspec.Parse` at load, as `review` and `fix` are.
The judge model stays the run-wide `judge` key.
Rationale: flat keys match `loom.yaml`'s existing shape and the strict loader's every-key-present gate; an empty value meaning "inherit" follows the `driver` and `friction` precedent.

### D5 — Fixer start option

Decision: `loom.yaml` gains `fix_start`, `parallel` (template default, today's behavior) or `after-review`.
With `after-review`, a round starts the fixer half only after the reviewer half has ended and Go has accepted the review and written the ready marker; the fixer's prompt is unchanged, and its first `lyx burler await-review` returns at once.
A reviewer half that ends without an accepted review starts no fixer, and the round reports the reviewer's outcome as it does today when the reviewer fails.
Any other value is refused at load, naming the two legal values.
The option is run-wide, applied to every review segment's rounds, and reaches `burlerengine` through `RunOpts`, so the engine never reads `loom.yaml` itself.
Bound: `after-review` only delays the fixer's start; it skips no gate, and the reviewer-before-fixer ordering of `PATTERN-review-round` is unchanged.
Rationale: the operator asked for the option only (2026-10-08); a run-wide key covers the cache-TTL case without per-segment surface nobody asked for.

### D6 — `loom resume` resumes a recorded circling decision

Decision: `lyx loom resume` over a run `awaiting` at a Bouncer row whose latest escalated round carries a recorded, not yet settled circling decision takes the same path as a halted run: it wakes the live, parked driver, whose step re-calls the Bouncer, which acts on the decision.
Its other refusals (no live driver, driver not parked, unfinished merge) apply unchanged.
Awaiting at a Bouncer row with no recorded decision refuses, naming `lyx loom circling accept` / `continue` and then `lyx loom resume`; awaiting at any other row keeps today's approve/reject refusal.
The circling verbs' `resume` output field becomes `lyx loom resume`, and the escalation brief's step 3, `internal/loomcli/circling.go`'s doc and every way-forward over an awaiting Bouncer row name `lyx loom resume` and the circling verbs, never approve or reject.
Bound: the new edge wakes a driver only for a Bouncer-row await whose decision file exists and is not settled; it gives no decision itself and never spawns a driver.
Rationale: issue #446: the verb named `lyx loom start` while the hub's role names `lyx loom resume`, and resume refused with PR-Gate verbs; a batten run leaves an escalated child with a parent notice to the driver's relay, so waking the parked driver is the step that moves the run and batten needs no change.

### D7 — PATTERN-review-round amended

Decision: `PATTERN-review-round`'s background bullet "a review segment converges only on a judge verdict over a fresh review, and fixed findings never converge on their own" is amended in the same commit as D1 and D2: a segment still converges only on a judge verdict, and from round 2 on a fix of a key first raised in the closing round may converge, carried to the decision record's `## Open risks`.
The PATTERN.md one-liner is reworded to match.
Rationale: D1 departs from the entry as written; PATTERN.md requires the change to be named and to land with the code.

### Added after Discussion (parent, 2026-10-08): D2 marker recognition

Decision: A carry-over marker is a line equal to the marker text after right-trimming, recognised only inside the ## Open risks section and outside fenced code blocks. The write refuses any marker set in that section other than zero or one well-formed pair per segment, where an opening marker is followed by a closing marker naming the same segment: an unclosed opening, an orphan closing, a duplicate pair for one segment and a closing marker naming another segment are all refused. An acceptance criterion pins that a record mentioning the marker text inline elsewhere (e.g. in ## Decisions) still settles.

Rationale: Round-3 F1: unspecified recognition made the refusal rule undecidable and would refuse the record that documents the feature.

### Added after Discussion (parent, 2026-10-08): D2 bound verified on the result

Decision: The write's bound is checked on the bytes it produced: re-read the new file and require that its sequence of H2 headings equals the prior file's, that each segment has at most one well-formed marker pair, and that every line outside the replaced span equals the prior file's. The prefix/suffix byte comparison against the slices the write cut is not named as the bound.

Rationale: Round-3 F2: comparing slices with the file they were cut from cannot fail.

## Constraints

- `PATTERN-review-round`: amended by D7; every other clause (review on disk before any target is touched, every finding fixed or disputed with evidence, reviewer and fixer separate sessions, the fixer touching nothing before the ready marker) holds, including under `fix_start: after-review`.
- `PATTERN-sole-parsers`: the carry-over write is a `discussionparser` function; ledgers and reviews are read by the code that reads them today.
- `PATTERN-config-strictness`: `loom.yaml`'s new keys load through the same strict path as its existing keys, and the template carries every key.
- `PATTERN-refusal-way-forward`: the new and changed `loom resume` refusals and the new config refusals name the way forward, with spec rows and a test reaching each refusal in loom.
- `PATTERN-no-denied-recovery`: no new way forward names a denied command.
- `PATTERN-shuttle-provider-seam`: the fixer start option lives in `burlerengine` and `loomengine`, never in `shuttleengine`.
- `PATTERN-stencil-ownership` and `PATTERN-producer-pointer-rule`: the judge stencil and escalation brief changes stay in their stencils; `decisionRuleMarker` stays the single holder of the rule's variants.
- `PATTERN-comment-line-breaks` and the repo's semantic-line-break rule apply to every touched comment and `.md` file.
- Docs land in the same commit: `doc.go` of `shedadapters`, `burlerengine`, `loomengine`, `loomcli` and `discussionparser` (whose doc and `append.go` header name `AppendDecision` as the one write path into the record) where their behavior changes, and `contracts/specs` rows for the changed refusals.

## Auto-mode assumptions

- Round 1 keeps today's rule (D1), although the board's proposal states the lighter rule without a round floor.
- The carry-over covers every open ledger entry at MEDIUM or worse, of any class, and every unlabelled open entry, not only fresh `design` keys, so a parent-accepted round carries its open findings the same way.
- The carry-over also runs on a recorded circling `accept` (D2).
- Webster-Review's carry-over goes to the decision record (D2).
- `fix_start` and the per-segment keys live in `loom.yaml`, not `burler.yaml`, since `burler.yaml` is a seed-only, operator-owned lens library and the models already live in `loom.yaml`.
- `review_circling_checkpoint` stays 3 after the budget drops to 3.

## Open risks

- From round 2 on, a fresh MEDIUM `design` fix that is wrong reaches the next producer unreviewed; the carry-over names it, and the guards are the stencils D2 changes: the Plan-Write or rework agent checks each listed Discussion-Review finding against the code, and the Describe item puts a Plan-Review or Webster-Review finding before the PR-Gate reviewer.
  Webster never reads the record, so a wrong Plan-Review fix is first checked at Describe, after Webster has built it; that is the accepted bound, and PR-Gate's rejection into rework is the way back.
  A wrong Discussion-Review fix that the plan writer does not catch becomes part of the Plan-Review answer key, which the Plan-Review rubric never questions.
- With `review_circling_checkpoint` and `review_max_bounces` both 3, CIRCLING never changes a default run's outcome; an operator who wants an early circling escalation lowers the checkpoint.
- The judge maps findings to keys; a judge that gives a finding already open in an earlier round a fresh key converges a round D1 means to continue.
  The only guard is the judge mapping against D1's earlier-open list and the previous ledger; no Go check reads the mapping.

## Acceptance criteria

- `decisionRuleMarker` for round 1 renders today's CONTINUE/CONVERGED rule; for round 2 and later it renders the lighter rule (BLOCKING, or MEDIUM-or-worse `design` on a key open in an earlier round); the CIRCLING bullet appears from the checkpoint on in both variants.
- The round facts list every key open in at least one earlier ledger, including at round 2 a key open in round 1 only.
- A missing or unparseable earlier ledger renders a parse-error line naming its round in the earlier-open list, and the round-2-or-later rule text says such a line means round 1's rule.
- The judge stencil carries no sentence that contradicts D1.
- A `CONVERGED` round whose ledger has open MEDIUM-or-worse or unlabelled entries writes one `## Open risks` entry between its segment's opening and closing marker lines, naming segment, round, closing path, each key with class, severity and the ledger rounds it was open in, and the anchor-relative review and fixer-report paths; every byte outside the entry is unchanged, and it still passes `lyx loom validate-discussion`.
- A key holding a line break, a backtick or a marker string is rendered on one line and cannot end the entry early.
- A skipped review leaves the segment's entry byte-identical.
- Each malformed marker set in `## Open risks` (an unclosed opening line, an unopened closing line, two pairs for one segment, a closing line naming another segment, a nested pair) is refused with an error naming the file, and the record is left unchanged.
- A record that spells the marker text inline in a sentence, in another section or in a fenced block settles normally, and that text is unchanged.
- The Plan-Write and rework stencils say a carry-over entry is findings to check, never scope; the Describe stencil turns each listed Plan-Review or Webster-Review finding into an item to check by hand; the Plan-Review rubric says a carry-over entry is not part of the answer key and licenses no scope.
- A `CONVERGED` round with no such ledger entry leaves a record that holds no entry for its segment byte-identical, and removes the segment's entry from one that does.
- A recorded circling `accept` writes the same kind of entry.
- A later generation's `Done` in the same segment replaces that segment's entry, and a replay of the same settle writes the same bytes.
- A failed write or commit returns an error and neither Approve nor Commit runs; a record with no `## Open risks` heading is left unchanged and refused the same way.
- The entry is committed by the end of the `Done` transition in Discussion-Review, Plan-Review and Webster-Review, through the carry-over seam's own commit of the decision record.
- The template's `review_max_bounces` is 3, and a CONTINUE at round 3 escalates with cause `budget`.
- Each of the six per-segment keys, when set, selects that segment's reviewer or fixer model per round; when empty, the segment uses `review` / `fix`; a malformed entry is refused at load naming the key.
- `fix_start: parallel` starts both halves as today; `fix_start: after-review` starts the fixer only after the review is accepted; a reviewer that fails under `after-review` starts no fixer; an unknown value is refused at load.
- `lyx loom resume` over a run awaiting at a Bouncer row with a pending circling decision and a parked live driver wakes the driver; with no decision it refuses naming the circling verbs and `lyx loom resume`; at PR-Gate it refuses as today.
- `lyx loom circling accept` / `continue` output `"resume": "lyx loom resume"`, and the escalation brief names `lyx loom resume`.
- `PATTERN.md` and `pattern/PATTERN-review-round.md` state the amended rule.

## Notes for the plan writer

- The Bouncer's settle path is `settle` → `settleUnconverged` / `settleUndecided` in `internal/shedadapters/bouncer.go`; the carry-over hooks in before Approve on both `Done` branches.
- Ledger entries with class and severity are already parsed for the judge facts (`bouncerfacts.go`); the round's review and fixer-report paths are known to the run directory helpers in `bouncerfiles.go`.
- `discussionparser.AppendDecision` in `internal/discussionparser/append.go` is the shape to follow for the `## Open risks` write.
- Model lists flow `loomengine.Config` → `shedrecipe.Env.ReviewModels` (`burlerengine.RoundModels`) → the BurlerRound row; the per-segment lists need the row's segment at that point.
- The fixer's start is in `burlerengine` `Engine` run (`roundHalves`, then two `StartGated` calls, then `join`).
- `loom resume`'s awaiting refusal is in `internal/loomcli/resume.go`; the circling decision reader is `readCirclingDecision` in `internal/shedadapters/circling.go`, and `loomrecipe.BouncerRunSubdir` resolves a row's run directory.
