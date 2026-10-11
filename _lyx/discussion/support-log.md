# Support log — burler-split-budget

## Interview

Autonomous run: no operator answered; every pick below is the writer's and is listed in the Question ledger.

1. Read the board entry and the entries it touches: `task-run-settings` (per-perch rounds as a task setting), `seats-replaces-singlellm`, `hold-deferral-end-on-held`, `kick-start-pack`, `fan-master-preread`, `parent-review-wait`, `orch-token-spend`.
2. Explored the loom and darn recipes, the loom config template, `shedengine`'s episode count and budget block, the Bouncer's budget seam, escalation and accept path, `BurlerProducer`'s hand-back rules, `MultiLLMProducer`'s outcome mapping, `burlerengine`'s doc and review parser, and `PATTERN-review-round`.
3. Routing: a Done from any segment row ends the Bouncer's episode, so the halves must hand off by Stuck; the empty-review skip lives in the fix row, which spawns nothing, rather than a new routing edge.
4. Producer type: `MultiLLMProducer` returns Done on a chair's success, so the rows are zero-advisor seat tables under the Burler adapter.
5. Darn's perch runs the same `BurlerRound` engine, so it splits too; it already has its own budget key and gets no auto-accept.
6. Lean fixer: dropping the rubric breaks the `Do not flag` dispute ground in `PATTERN-review-round`, so D9 amends it; the review gains a required `fix` key so the fixer has its instruction.
7. Budget: per-perch override keys over the run-wide ones; auto-accept reuses the existing accept path, for the budget cause only.
8. Lens review: six forks spawned before the files were written; all six returned. One line per merged finding:
   - premises BLOCKING, reviewer gate failure as a counted Stuck contradicts `decideRound` and `BurlerProducer`'s seed-call rule: fixed, D3 keeps it a hard error, D4's no-acceptance case is a hard error with a goto way forward.
   - premises MEDIUM, review-bytes guard lost across rows: fixed, D3 stores a digest in the acceptance record and D4 re-checks it.
   - premises MEDIUM, acceptance record vs orphan review semantics: fixed in D3, with a restart criterion.
   - scenarios MEDIUM, fix-gate exhaustion and archive rule unplaced: fixed, D4 bullet and criterion.
   - scenarios MEDIUM, unaccepted review on restart and crash before the record: fixed, D3 orphan rule and restart criterion.
   - scenarios MEDIUM, hub target N=1 not a scenario: fixed, D10 line and criterion.
   - pattern-fit MEDIUM, both halves would share `burler-chair`: fixed, D2 role prefixes and the `PATTERN-agent-name` constraint reworded.
   - pattern-fit MEDIUM, budget refusal spec row unnamed: fixed, D10 and Constraints.
   - pattern-fit, goal-scope, scenarios and leanness (merged), retired-row refusal criterion licensed by no Decision: fixed, criterion dropped; the restart rule stays in Constraints.
   - goal-scope MEDIUM, board's open question on the channel skipped: fixed, D8 cites the fork's transcript search (no `SendMessage` to a `burler-advisor`); Q7 updated.
   - goal-scope LOW, D4 vs board "skips Burler-Fix": fixed, D4 says the row runs but spawns no session.
   - goal-scope LOW, model keys after the split unstated: fixed in D10.
   - goal-scope and board-fit LOW, `hold-deferral-end-on-held` unmentioned: fixed, Out line.
   - board-fit LOW, D2 overstates `seats-replaces-singlellm`: fixed, D2 rationale and Out line.
   - board-fit LOW, fan and kick-start notes land on the review row: fixed, Out line.
   - board-fit LOW, `parent-review-wait` cites a removed fixer wait as precedent: rejected, that note's own text is the board's to fix; this record changes nothing it needs.
   - pattern-fit LOW, loomshed name constants: fixed in D1.
   - pattern-fit LOW, `premise false` still names the fasit: fixed in D9.
   - pattern-fit LOW, auto-accept distinguishability vs carry-over class: fixed, D11 says the decision file only.
   - premises LOW, second `Routing` caller in `arm.go`: fixed in Notes.
   - premises LOW, double-counted failure: rejected as moot, a review gate failure is no longer a Stuck.
   - scenarios LOW, seam failure with auto-accept: fixed in D11's bound and a criterion.
   - scenarios LOW, required `fix` breaks prior-round parsing: fixed, D7 enforces it at the current round's gate only.
   - scenarios LOW, flag flipped after a recorded accept: fixed in D11.
   - leanness LOW, repeated assumptions, stale figures in D6, padding constraints, D10 placement hint: fixed by trimming.
   - leanness note, the skip report must parse: fixed in D4 and a criterion.

## Rejected alternatives

- One seat table with the chair's spawn deferred until the advisor delivers: special code in `seatengine` (board entry, operator).
- The fixer as a fork of the reviewer: the fixer inherits the whole exploration (board entry, operator).
- Burler-Review returning Done on an empty review straight to the Bouncer: a Done ends the segment's episode and resets the Bouncer's budget, so a judge that keeps saying CONTINUE over clean reviews would loop without bound.
- A third routing edge in `shedengine` for the skip: a new FSM concept for one producer, where a no-spawn fix row does the same.
- Using `MultiLLMProducer` itself for each row: its Done on success contradicts D1; making its outcome mapping configurable adds options to the generic producer for Burler alone.
- Replacing `review_max_bounces` with per-perch keys outright: breaks every existing `loom.yaml`; overrides match the per-segment model keys.
- Per-perch circling checkpoints: nobody asked for them; the template comment states the relation per perch.
- Auto-accept on the circling cause too: the board asks for a spent budget only, and circling is the judge flagging a loop a human should see.
- Keeping the reviewer alive for fixer questions: no live reviewer exists between sequential rows, and no archived round depended on one.
- Darn auto-accept: not asked for.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): What runs each row? Zero-advisor seat table under the Burler adapter, not `MultiLLMProducer`; rationale D2.
- Q2 (auto-pick): How is the fixer skipped? The fix row spawns nothing and writes a Go-authored fixer report; rationale D4.
- Q3 (auto-pick): What counts as empty? Zero findings of any severity; `PATTERN-review-round` fixes every severity.
- Q4 (auto-pick): Does darn's perch split? Yes, one engine; no darn auto-accept.
- Q5 (auto-pick): What does the lean fixer get? Review, named targets, write surface, commit and dispute rules; not rubric, fasit, PATTERN overview, focus, prior rounds.
- Q6 (auto-pick): How does the review carry the fix? A required `fix` key per finding, enforced by the review parser.
- Q7 (board open question, auto-pick): Does the fixer use the reviewer channel? Dropped; a lens fork's search of local transcripts found no `SendMessage` to a `burler-advisor`, and the board's 31-round evidence shows every finding applied.
- Q11 (auto-pick, lens review): What does a reviewer gate failure do? Stays a hard error, archived, as today.
- Q12 (auto-pick, lens review): How does the fix row trust the review? The acceptance record carries a digest, re-checked after the fixer.
- Q8 (auto-pick): Per-perch keys shape? `<perch>_max_bounces` (0 inherits) and `<perch>_auto_accept` (false).
- Q9 (auto-pick): Auto-accept scope? Budget cause only; an already recorded escalation stands.
- Q10 (board open, resolved by operator in the entry): only verify checks Webster fixes after one auto-accepted round; accepted.
