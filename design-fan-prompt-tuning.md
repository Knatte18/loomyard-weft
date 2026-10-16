# Burler lens fan and twill fixer in production, per-segment lenses for Discussion and Plan review

`fan-prompt-tuning` · Task · enhancement, burler, stencil, loom · running · Webster-Burler-Review

## Idea (operator, 2026-10-13)

The Burler lens fan is parked because the fan-lab bench found it no better than a solo round, and its forks found mostly overlapping findings.
The operator thinks the fan could pay with better-tuned prompts: lenses and fork instructions that send each fork after different defects.

## Starting point

- The operator's first bench of Burler's fan, when it was designed, found a fair amount, though mostly less advanced defects; twill, built after it, proved clearly more effective (operator, 2026-10-13).
  Twill's forks share the writer's context and their findings are judged by the writer, which may be the design to borrow.
- Bench results: `docs/research/lens-fan-lab-phase1.md` ("Conclusion") and `lens-fan-lab-phase0.md`, cost in `lens-fan-cost-analysis.md`.
- On Discussion-Review a `sonnet[medium]` fan missed the BLOCKING defect that solo `opus[medium]` found; a `sonnet[high]` fan found it but passed 300k context and cost more than solo.
- On Plan-Review the `sonnet[high]` fan's surplus was mostly LOW findings, at about six times a solo round's cost.
- Variant D's master manifest (pre-read without judging) is the report's named starting point if the fan is revisited.
- Forks inherit the reviewer session's model (`fan_review`), so master and forks cannot run on different models.

## Scope to discuss

- Lens texts that overlap less: what each lens owns, and what it must leave to the others.
- Measure overlap per lens against the solo round's findings, on the fan-lab fixtures, before any hub setting changes.
- Fan on or off per round (for instance fan in round 1 only) needs code; see `model-downgrade` for per-round models.

# Merged from note `fan-master-preread` (2026-10-15)

Both notes tune the parked lens fan's prompts in `internal/burlerengine/prompt.go`.

## Seen

The lens fan's design: the master reviewer reads most of what the round needs first, then forks, so each lens fork inherits that reading through the cache and needs little new reading, which keeps a fork cheap enough for a Sonnet medium reviewer.
The first fanned run (`webster-replan-state`, Discussion rounds 1-4 and Plan round 1) did not work that way; the analysis is in `docs/research/lens-fan-cost-analysis.md`:

- The master forked after reading only the target, instructions and stencil (39-74k context), no code, then told every fork to verify the Decisions' premises against the code.
  In Discussion round 1 the master read 19 code files first, and its forks took 32 turns and 152k fresh tokens; rounds 2-4 took 81-125 fork turns and 435-485k fresh.
- Inheritance through the cache works; cost scales with fork turns, about 3.5-4k fresh tokens per turn.
  32-46% of fork file reads duplicated another fork's.
- Fanned Discussion rounds cost 301-638k fresh against 86-121k for solo Opus rounds; Plan round 1 cost 901k fanned against 202-227k solo.
  The fan raised more MEDIUM and LOW findings, not more BLOCKING ones, and about half of raw fork findings were duplicates across lenses.

The responsible text is `clusterRulesBlock` in `internal/burlerengine/prompt.go`: "After exploring the target fully" never says to read the code the target cites, and "Fetch only what your lens needs" sets no budget.
The `standard` fan also points code lenses at design documents.

## Idea

Revise the fan so it works as designed:

- the master is told every lens of the fan up front, and before it spawns any fork it explores, for each lens, what that lens will need and reads it, so the forks inherit one shared reading instead of each exploring on its own (lyx cannot pick the relevant code itself: Discussion review is exploratory, so the master's judgment chooses what to read);
- the master judges nothing before the fork point: a review written before forking would colour every fork, so the master's own holistic review stays after the forks are spawned;
- each fork is told its context already holds the code, and gets a tool-call budget (about 6) for reading only what is missing; forks spending their budget show in the usage record that the master read too little;
- Discussion-Review and Plan-Review use the `discussion` and `plan` fans instead of `standard`.

Estimate: about 200-250k fresh tokens per round, about 2x a solo Opus round instead of 5-6x.

## Operator decisions (2026-10-10)

- The fan stays off in the hub until this task lands and a fanned run shows the estimate holds.
- Until then Discussion-Review and Plan-Review run solo on `opus[medium]`, and Webster-Review on `sonnet[medium]`.

## Research (fan-lab, 2026-10-10)

A research agent replayed `webster-replan-state` Discussion round 2 standalone (`lyx burler run`) per variant, master and forks `sonnet[medium]`; the full reports are `docs/research/lens-fan-lab-phase0.md` and `lens-fan-lab-phase1.md`.
Variants: A today's text; B a lens-agnostic reading checklist in step 1; C B plus a 6-call fork budget; D C plus a reading manifest the master writes before spawning.

| Run | Fan | Master code pre-reads | Fork tool calls | Review side fresh | Verdict | F1 (BLOCKING) |
|---|---|---|---|---|---|---|
| A | discussion | 0 | 2 | 216k | APPROVED | no |
| B | discussion | 2 | 8 | 154k | APPROVED | no |
| C | discussion | 2 | 0 | 133k | APPROVED | no |
| D | discussion | 12 | 1 | 198k | APPROVED | no |
| D | standard | 3 | 1 | 154k | APPROVED | no |
| original round 2 | standard, sonnet[high] | 0 | 135 | 619k | BLOCKING | yes |

Findings:

- Only the manifest (D) makes a Sonnet-medium master pre-read; in A-C it read instruction 2 at once and forked after the board entry and a few pattern files.
- No master judged anything before the spawn, in any variant.
- Only the D runs reproduced round 2's code-grounded MEDIUMs; A-C findings were text-level.
- No run found F1, which needs one hop past the target's citations (the D7 guard on `push` → `Fabric.Commit` → `SpawnDetachedPush`); at medium effort the forks barely read in any variant, so no one made the hop.
- Fork cost is driven by effort, not the budget: the original's 485k fork fresh came from `sonnet[high]` forks told to verify against code.
- A D round costs about 150-200k review side, still 1.5-2x a solo `opus[medium]` round.
- Every fork already inherits instruction 2 and so every lens, since the master reads it before spawning.

Recommendation: adopt D, with a checklist line for callers and spawned processes (untested). Step 1 (`clusterPrereadBlock`, explore file only, cluster rounds only):

```
## Reading for the fork reviewers

This round fans out: in step 2 you spawn fork reviewers, and each fork inherits your context as it stands at the spawn.
Your reading in this step is the forks' reading, so read here what the review will need, once, instead of leaving each fork to read it again.

Before you leave this step, read:

- every file the target names, at the definitions of the symbols it cites;
- for each symbol the target changes, guards or removes: its direct callers and callees, and every path that reaches it from another process, such as a child the code spawns or a command that calls the guarded verb in-process;
- the `pattern/` background file of every PATTERN entry the target names or touches;
- the board entry of the task the target belongs to, through `lyx board get`;
- the failure, restart and resume paths of each change the target makes.

Read definitions and call sites, not whole files: a fork re-reads its inherited context on every turn.
Judge nothing while you read: write no finding, verdict or assessment, not even in a status line or a note to yourself.
A judgement in your context before the spawn colours every fork; your own review starts after the forks are spawned.

End this step with a reading manifest as a plain message: one line per file and symbol you read, grouped by the part of the target it serves.
The manifest names what you read and nothing about what you found.
```

Step 2 (`clusterRulesBlock`), replacing the spawn trigger, the fork boilerplate and the holistic paragraph; the consolidation paragraph is unchanged:

```
This is a cluster round: as soon as step 1's reading is done, spawn ALL of the fork reviewers below in a SINGLE message via the Agent tool with `subagent_type: "fork"`, one per lens listed here — never pass a `name` (named forks silently lose inherited context).

Lenses for this round:

<lenses>

Each fork's prompt is this boilerplate plus that lens's emphasis text, and nothing else: never add a direction to verify against the code or to read files.
Your inherited context already holds the target, the rules and the code the target rests on; review from it.
The reading manifest in your context lists what is already read; check it before any tool call.
You may make at most 6 tool calls, each only for something your lens needs that the manifest lacks, such as a caller or a spawned process one hop beyond it, and never to re-read what your context already shows.
End your final message with one line per tool call you made: what you read and why your context lacked it.
You are READ-ONLY — never Write/Edit/delete any file, never run any git command, never touch the two round output files, and never call the Agent tool yourself (forks cannot nest); return your findings ONLY as your final message, each with a severity, a class, a location, and a one-line summary.

While the forks run, YOU (the handler) do your own HOLISTIC review — architecture, cross-file invariants, PATTERN-fit — from the same context, reading further only where it or a fork's read list shows a gap, and prepare the ground truths and the severity rubric you will judge every finding against.
```

## Confirming run (2026-10-10): parked

The recommended text on the standard fan at `sonnet[high]` aborted at the 300k cap with no review written: review side 354k, $3.19, 288 s active.
Three forks raised F1 through their hop budget, so the pre-read and the hop line work at high effort, but the round is slower than solo `opus[medium]` (225 s, $1.45, found F1) and about twice the price.
Solo `opus[medium]` stays the reviewer; this entry is parked as a note.
Two hand-built packs were then tried on solo `opus[medium]`, one run each:

- A body dump (the bodies of the 35 symbols the target cites, about 20k tokens, in a file read first): 312 s, $1.93 review side, missed the BLOCKING finding.
- A kick-start pack in quarry's `RenderKickstartPack` format (a glyph→span line and a signature line per glyph, 35 glyphs, 5.3 KB), injected into the first prompt with "read all listed spans in parallel, in one turn, before doing anything else": 316 s, $1.94 review side, found the BLOCKING finding (1 BLOCKING / 4 MEDIUM).

The operator doubts the second run applied the kick-start as designed either, and will look at it later.
Plain solo stays faster and cheaper for review on this fixture; the kick-start pack's measured case remains the implementer and the Burler fixer (`kick-start-pack`).

Open if the fan comes back:

- The landing text must say "go straight on to step 2 in the same turn": the master ended its turn after posting the manifest.
- The `discussion` fan has no lens aimed at a guard's callers; `scenarios` is the natural owner if a lens change is wanted.
- Proposal E (not run): lyx extracts the target's citations (paths, backticked symbols) as the manifest's starting list without loading code, leaving the choice of what to read to the master; 22 of the 25 files the original forks needed are cited in the target, the other 3 one hop away.

## Operator view, not a decision (2026-10-16)

The operator believes fan-out reviewers can work: they need better tuning and harder thinking, and then one reviewer run yields more findings.
The aim is broader reviews from the Burler reviewer's lenses: the earlier tuning attempt had the lenses find mostly the same, and the operator wants another tuning attempt in time.
Related: `perch-review-analysis` (issue #676) measures rounds and late BLOCKING findings and proposes two parallel reviewers in round 1.

The operator's plan for the tuning, stated 2026-10-16 (not yet a decision):
- The tuning is a bench, not a task: an iterative bench that tunes the lenses and the rest of the reviewer prompt until the fan is useful.
- The bench agent takes one review and tunes the lenses over a few iterations, up to 3 rounds at first.
- The first lens set must be based on something, not invented.
- Too many lenses are not useful: perhaps 3-4.

## Bench result (2026-10-16)

The lens-fan tuning bench, cut to a proof of concept at the operator's word, is in `docs/research/burler-fan-tuning-bench-2026-10-16/report.md`.
On `reed-view` Discussion-Review round 1, a 4-lens fan found 16 distinct real defects (8 at MEDIUM or above) against 11 and 9 (5 and 6) for two solo runs, including the round-4 BLOCKING both solo runs missed, at about 1.9x a solo round's cost and the same wall time.
The `existing-paths` lens found the late BLOCKING alone; `way-forward` was the noisiest.

## Operator decision (2026-10-16)

The lens fan goes into production:
- The lens set is configured per review segment.
- Discussion-Review gets the lenses `interactions`, `existing-paths` (widened to the readers of a changed state), `bound-breaker` and a walkthrough lens in place of `way-forward` (execute each decision's mechanism step by step on a concrete case, with the data each step is handed, against the decision's stated purpose), plus the bench's prompt additions: the explore step's pre-read manifest for the fork reviewers and the review step's fork rules, with the off-lens BLOCKING escape dropped.
- Plan-Review gets its own lenses, based on the Plan findings in `perch-review-analysis`.
- One task builds it; real runs are the reference from here, with no further bench.
The bench's lens texts and stencil additions are in the report's `setup/` folder.

# Merged from note `burler-fixer-fork` (2026-10-16)

## Seen

Carried over from the removed note `model-downgrade` (2026-10-15).
Each Burler fixer is a fresh session that reads the whole context the reviewer just read, part of the $1.9 per run in the Burler cost breakdown.
Skipping the fixer on an empty review is already built (`internal/burlerengine/fix.go`).

## Idea

Fork the fixer from the reviewer's session, so it inherits the reviewer's cache prefix instead of reading the context again.
A fork inherits the reviewer's model, so this rules out a separate fixer model for a forked fixer.

## Other open items from `model-downgrade`

- Plan and Discussion fixers to `sonnet[medium]`: estimated about $4 over six runs, not benched.
- Fan on or off per round (fan in round 1, solo after) does not exist and needs code.
- A per-batch Webster model (a cheap model only for mechanical cards) needs code in `internal/websterengine`.

## Operator view, not a decision (2026-10-16)

The operator wants a far more critical fixer that uses the Twill setup to review its own work before the next review round.
`perch-review-analysis` (issue #676) proposes the same: after its fix the fixer spawns forks of itself that check consistency, claims and the way forward, aimed at late findings an earlier fix caused.
This pulls against this note's cost idea, since a Twill fixer spends more tokens per round to need fewer rounds.

## Operator proposal, not a decision (2026-10-16): reviewer master that forks

Taken from an agent's suggestion in a discussion on another machine.
The Burler reviewer explores the code as today, and just before it reasons out its review it forks into N review forks, one per lens.
N is 1 for now, since no fan-out is on: one general lens, the review we have today, but lenses must be supported.
The master waits; when the forks are done it gets their reviews and fixes the findings critically.
Then the master forks again, and the forks give a Twill-like critical review of the fix itself, much like the Twill self-review the Discussion and Plan writers do today.
The fix itself may also be done by a fork, so the master does not hold the reasoning behind the fix either.
The gain the other agent pointed out: the reviewer master already has every file in context, so no fresh fixer reads it all again, which saves tokens and time.
A smaller first step: keep today's reviewer and fixer split, and make only the self-review of the fix part of the fixer agent.
The operator asked for an audit of whether this is feasible and worth it (2026-10-16).

## Audit result (2026-10-16)

Report: `docs/research/burler-fork-review-audit-2026-10-16/report.md`.

- A fresh fixer re-reads little: about $0.40 and 1 minute a session, all fixers 7.3 % of run cost over the last 10 runs, a re-read of 15-35k tokens.
- A reviewer master that fixes (stages 1-2) is about break-even on Discussion and dearer on Plan at its 90-140k end context, needs a PATTERN change (reviewer and fixer are separate sessions) and gives up a separate fixer model; with one lens the review fork is pure overhead.
- A master that forks to review its own fix (stage 3) costs about $0.3 a round more than the same check inside today's fixer, since its forks inherit 100-250k instead of 40-65k.
- The fix in a fork (stage 4) is feasible, but the Twill writer-fork benches showed no gain.
- Stage 0, the agent's recommendation: today's fixer fixes, then forks K read-only copies of itself, each checking one aspect of the fix (consistency of siblings, claims against the code, way forward can be carried out), and folds their findings in before it writes its fixer-report.
  Estimated +$0.55-0.75 and +2 minutes a fixer session; it pays if it saves one round per 2.5-4 self-reviewed fixes, and it checks the closing round's fix, which nothing reviews today.
- Independence: of about 630 findings in 60 runs, fixers disputed 1 and fixed about 2 % differently; the fresh reviewer each round is the real gate and stays.
- Only in-session fork subagents share the cache; an engine without them falls back to today's two-session round.

## Operator view, not a decision (2026-10-16)

- A bench should test whether a Twill fixer in Burler is useful.
- The fixer mode must be selectable as `single` or `twill`, as the Discussion and Plan writers' already is, and per Burler perch: Twill for Discussion and Plan, probably not needed for code.
- Since the fixer now also becomes a kind of reviewer, its context may grow a lot; the bench will show.

## Twill fixer bench result (2026-10-16)

Report: `docs/research/burler-twill-fixer-bench-2026-10-16/report.md`, a proof of concept over three round-1 fixes (two Plan, one Discussion), one run per arm.
- The twill fixer ran as designed: three read-only forks in one message, 1-2 reads each, no wrong fork finding.
- Fix-introduced findings 3 against 8 for today's fixer, wrong dispositions 1 against 3: a lead, not a measured effect.
- Cost +$0.71 and +1 minute a fixer session; fixer end context 85k against 63k.
- Every fix-introduced defect left in the twill arm came from the reviewer's own proposed fix, and twice the forks read the flawed text and endorsed it: the forks check the execution of a proposal, not the proposal.
- Rounds saved are not shown; real runs are the reference.

## Operator decision (2026-10-16): twill fixer, coordinated with the reviewer lenses

- The Burler fixer gets a mode, `single` or `twill`, set per perch in `loom.yaml`; the default is `single`, and Discussion and Plan are set to `twill`.
- The twill fixer's self-check forks follow the bench's design (`setup/selfcheck-block.md` and `setup/fix-check-rubric.md` in the report folder): after its last fix the fixer spawns read-only forks, judges their findings and records them in a `## Self-check` section of its fixer-report.
- The reviewer's lens set and the fixer's self-check lens set are designed together per perch so they complement each other, not repeat each other.
- A defect in the reviewer's own proposed fix is caught at reviewer level or by a lens that challenges the proposal itself (whether the proposed change is right and what it breaks), not only its execution.
- This is built in this task together with the lens fan; real runs are the reference, with no further bench.

## Operator view, not a decision (2026-10-16)

The aim of the two together: the reviewer's lens fan and the twill fixer make one Burler round's review and fix much more solid, so fewer rounds are needed.

# Moved from note `perch-review-analysis` (issue #676, 2026-10-16)

The late BLOCKING findings per perch, the evidence the Plan-Review and Discussion-Review lenses are based on.

## BLOCKING findings do not converge to zero

Share of review rounds with at least one BLOCKING finding:

| Perch | Round 1 | Round 2 | Round 3 | Round 4+ |
|---|---|---|---|---|
| Discussion | 51 % | 29 % | 24 % | 23 % |
| Plan | 42 % | 36 % | 38 % | 33 % |
| Webster | 21 % | 49 % | 37 % | 40 % |

The rate drops after round 1 and then flattens at 25–40 %, so a hard cap at 3 would cut real findings unless the late ones are noise.

### What the late BLOCKING findings are

Every BLOCKING finding in rounds 4 and later (31 findings across 15 runs) was classified against the artifact history and fixer reports:

| Perch | A: introduced by an earlier fix | B: there from the start, missed | C: not really blocking | Total |
|---|---|---|---|---|
| Discussion | 4 | 3 | 0 | 7 |
| Plan | 1 | 3 | 0 | 4 |
| Webster | 0 | 7 | 13 | 20 |

- **Discussion:** all late findings are real design holes, and most are consistency regressions from fixes.
  A fixer replaced one false claim with another (shuttle-state, round 3 → 4), or changed D6 without updating the sibling D7 it now contradicted (engine-model-choice).
- **Plan:** the holes were in the first draft, and each fix covered only part of the affected set.
  In config-layers-run-settings, round 4 found four stale clone tests under one key and round 5 found three more under the same key.
- **Webster:** 13 of 20 were comment or markdown line-break style rated BLOCKING, about 7 of them on lines a fixer had just re-wrapped at the reviewer's request, so the loop generated its own findings.
  The real behaviour bugs (gate-test-scope F1, reed-multi-window F1/F2) were there from the start, and the reed ones surfaced only when real tmux ran at publish, because earlier rounds ran fake tests only.
- **Across perches:** in most runs the round-4 fix was never reviewed again, because the escalation was accepted at the budget or hard cap.

Webster A vs B rests on fixer reports and review texts, not code diffs: the per-round warp commits are gone after the squash merges.
The full table is below.


<details>
<summary>Full classification of the late BLOCKING findings</summary>

## Discussion

| perch | round | slug | finding title | class | justification | bouncer verdict | resolved next round? |
|---|---|---|---|---|---|---|---|
| discussion | 4 | config-layers-run-settings | `loom.discussion_producer` opens a review switch-off to the local layer | A | The targeted sentence ("keeps `discussion_producer` open because it picks the writer, not a reviewer") was added by the round-3 fixer (round-3-fixer-report.md:12), following round-3 review's own claim (round-3-review.md:58) that the key acts on the writer. The key being open is part of the recurring LocalKeys-enumeration defect, but the false premise itself is a round-3 fix product. | CIRCLING | No round-5 review; round-4 fixer added the key to LocalDenied, escalation accepted (cause budget) and the parent appended a D4 addendum making it LocalDenied. |
| discussion | 4 | engine-model-choice | D7's wiring refusal contradicts D6's no-key-check rule | A | D7's "refused at wiring" sentence is in the initial record (633c2f347 line 108) and was never touched, but the D6 rule it contradicts (`record-batch`, `reset` etc. check no key; checks at spawn sites) was written by the round-2 and round-3 fixers (round-2-fixer-report.md:13, round-3-fixer-report.md:10). The inconsistency is a consistency regression from those D6 fixes not propagating to D7. | CONTINUE | No round-5 review; round-4 fixer made D7's wiring refuse nothing (round-4-fixer-report.md:6), then escalation accepted at hard cap. |
| discussion | 4 | logging-cleanup | D17's ready-task fallback starts any ready task, unbounded | A | The "starts any task the board shows ready and not yet started" clause was introduced by the round-2 fixer (round-2-fixer-report.md:5) and decoupled from the deploy-gated wake by the round-3 fixer (round-3-fixer-report.md:6), which made it unbounded. Not in the initial record. | CIRCLING | No round-5 review; round-4 fixer bounded it to the notice-log `newly ready` line, then escalation accepted at hard cap. |
| discussion | 4 | loom-bouncer-rounds | D4 budget-reached test rests on a budget the seam does not report | B | The core defect (D4 reads the "segment's" count through the `Bounces` seam while D3 drops the literal `max_bounces` so the seam's budget falls to `defaultMaxBounces`) is in the initial record (63b1e694a D3/D4 lines 62-73); rounds 1-3 flagged only the off-by-one, never the seam's budget source. The round-3 fix made the `review_max_bounces` comparison explicit, which sharpened but did not create the mismatch. | CIRCLING | Yes: round-4 fixer stated the test as the seam's `count >= budget` and required the seam's routing to go through the config assembly; round 5 CONVERGED and confirms the D4 budget test. |
| discussion | 4 | reed-view | Zoom does not survive reed's own resize and apply paths | B | D1's zoom (`resize-pane -Z`) and D7's `window-resized` pin hook are both in the first decision-record commit cb84bc254, and rounds 1-3 reviewed D7's pin arithmetic three times without checking the pins against zoom. Fixers 1-3 refined pin heights but did not create the conflict. | CONTINUE | Yes: fixer 4 added re-zoom around every geometry step plus a Bound and criterion; round 5 has only MEDIUM/LOW follow-ups on D1 (pane order while zoomed, other unzooming commands). |
| discussion | 4 | shuttle-state | D1 bound on line corruption is false for the rewritten Stop and ask hooks | A | The false sentence ("as today's pair can … adds no new way to corrupt a line and closes none") was written by fixer 3 (F2: "D1 drops the one-write claim and states that a large line is flushed in chunks and can interleave, as today's two-write chain can") in answer to round 3's MEDIUM on the same point. The original one-write claim was there from the start, but round 4 targets the fixer's replacement wording. | CONTINUE | Yes: fixer 4 kept the payload on today's `cat` command so the Bound holds; round 5 has no D1 write-bound finding (the bouncer judged the recurring key not a defect). |
| discussion | 4 | webster-replan-state | D8 git forms accept quoted or escaped options | B | The first D8 (commit with D8 at line 90) already admitted `git log`/`ls-remote` with arbitrary words, so the quoting bypass was there from the start. Rounds 1-3 flagged write-capable flags each time (key `d8-reader-write-flags-unbounded`), and each fixer narrowed the flag list without closing the quote/escape path, so the bound stayed false. This is an incomplete fix of a real recurring finding, not a regression. | CIRCLING | Fixed by fixer 4 (all words plain, new rejection rows); the round-4 circling decision was "accept, cause: budget"; there is no round-5 review to confirm it. |

## Plan

| perch | round | slug | finding title | class | justification | bouncer verdict | resolved next round? |
|---|---|---|---|---|---|---|---|
| plan | 4 | config-layers-run-settings | Card 15 leaves the tests that expect the old clone output unplanned | B | "In a hub nothing is ever added, and an absent one stays absent" is in card 15's first draft (commit 27b22d543, line 44), so the clone's empty prime and the stale tests existed before round 1. Rounds 1-3 reviewed card 15's clone path (R2 F1 seeding, R3 MEDIUM clone outcome) without listing the tests that read the prime's per-worktree files. | CONTINUE (circling decision: continue, cause budget) | Partly: the four named tests were added to card 15, but round 5 found three more tests under the same key (CIRCLING). |
| plan | 5 | config-layers-run-settings | Clone tests that read the prime's or a pair's per-worktree files are not carried | B | It is the same enumeration gap from card 15's first draft: `TestRunCLI_CloneEndToEnd`, `AnchoredHubScenario` and `PairInheritsConfigs` were affected from the start, and the round-4 fix enumerated only part of the set. It was an incomplete fix, not a new defect brought in by one. | CIRCLING (circling decision: accept, cause budget, settled) | No round 6; the round-5 fixer added the three tests and reran the search, but no later review checked the fix. |
| plan | 4 | logging-cleanup | Card 9's decision-read guard rows are unreachable, and one guarded site never repeats | B | The guard targets and rows for `handedLockStep`, `parentMovedUnderGate` and `undecided` are in the first draft of the card (then 08-battenshed-read-guards, commit 64956eb89), and no fixer report from rounds 1-3 touched them. Round 1 even listed card 8 as one of the cards that "reach all three branches", which misses the hard end in `awaitingStep`. | CONTINUE (circling decision: accept, cause hard-cap) | The round-4 fixer fixed it (handedLockStep moved to keep, pause-covered rows), but no review checked the fix: the run stopped at the hard cap. |
| plan | 4 | webster-dead-ends | Archive-only reset with pending findings has no way forward that makes progress | A | The round-3 fixer (F4) gave `headBeforeEveryStart` and HEAD-not-at-start the reset caller step `lyx webster accept-audit`, which is exactly the step round 4 shows cannot succeed; that F4 fix applied the round-3 reviewer's own suggested fix, built on round-2 F3, which put the pending-findings guard on the reset. The bouncer also notes it "originates in the round-3 fix". | CONTINUE (circling decision: accept, cause budget, settled) | The round-4 fixer fixed it (new decision `reset-guard-judges-against-head`), but no round-5 review checked the fix (only a focus file). |

## Webster (bolt-and-store, bugfix-fabric, bugfix-sessions, gate-test-scope)

| perch | round | slug | finding title | class | real-bug or test/doc | justification | bouncer verdict | resolved next round? |
|---|---|---|---|---|---|---|---|---|
| webster | 4 | bolt-and-store | F1 PATTERN.md exceeds the overview size cap (16430 > 16384) | B (latent, merge-induced) | doc-size cap / failing pattern test | The card 12 hub-store-gc line growth was in the original implementation, and no fixer in rounds 1-3 touched PATTERN.md. The file only went over the cap after the Publish merge-in of main (6f3760049), so earlier reviews could not have seen it fail. | CONTINUE | Yes: r4 fixer a0251d2d6 brought it to 16382 bytes, and r5 confirmed it (CONVERGED). |
| webster | 4 | bugfix-fabric | F1 switchOrForkWeft doc line 137 joins two independent clauses | C (origin A, r2) | comment style | The r2 review asked for checkout.go:137-138 to be joined onto one line, and fixer r2 did that, which created the comma-plus-so line that r4 flags. Only PATTERN-comment-line-breaks makes it gating; no code defect was found in any round. | CIRCLING | No r5 review: the r4 fixer applied the fix (fc71fd002), and the circling decision was "accept", settled. |
| webster | 4 | bugfix-fabric | F2 remoteleftover.go header line 2 and leftoverKind line 20 hold two clauses | C (origin A, r3) | comment style | The r3 review flagged remoteleftover.go:2-3 as a break inside a compound predicate, and fixer r3 put both predicates on line 2, which produced the "so Add adopts it" join (the bouncer notes this). Line 20 sits in a file that fixer r2's F3 pass also edited. | CIRCLING | Fix applied (841df36cd), but there is no r5 review; the run was accepted at circling. |
| webster | 4 | bugfix-fabric | F3 Deps.ConfigChanges/Notify docs break a compound predicate and keep a clause joined | C (origin A, r2) | comment style | The r2 review cited deps.go:126 as a "same shape recurs" site, and fixer r2's F3 break "after the comma before the conjunction" split the compound predicate across 126-127. The line 138 join was probably original text (B). | CIRCLING | Fix applied (c63604a6d), but there is no r5 review; the run was accepted at circling. |
| webster | 4 | bugfix-sessions | F1 Comment lines that end mid-phrase (4 sites) | C (origin mixed A/B) | comment style (one site is a test comment) | entryobservation.go:62 sits in the sentence fixer r3 "rewrote whole", so that site is A. seed.go:136 and smoke_test.go:640 are one-phrase substitutions in legacy paragraphs that the original diff changed and no fixer touched, so they are B. The finding is purely about line-break style. | CIRCLING | Fix applied (1d6e032a7), but there is no r5 review; the run was accepted at circling. |
| webster | 4 | bugfix-sessions | F2 Two independent clauses on one comment line (5 sites) | C (origin mostly A, r2/r3) | comment style | Fixer r2 rewrote the claudeengine.go header ("methods are pure functions"), giving the :3 site. Fixer r3 rewrote the integration_driverbootstrap_test.go sentence and broke orchengine/doc.go, giving the :22 and doc.go:231/233 sites. The orchcli/doc.go parenthetical is unclear. | CIRCLING | Fix applied (0887623b7), but there is no r5 review; the run was accepted at circling. |
| webster | 4 | bugfix-sessions | F3 Serial untagged tests edited by more than one line lack t.Parallel | C (severity inflation; origin B) | test-only hygiene | The ten tests were edited by the original diff, and no fixer touched them (fixers r2/r3 parallelized other lists). The same defect class was rated LOW in r3 (F5, watcher_test.go:910), and r4 raises it to BLOCKING by widening the rule to "edited by more than one line". | CIRCLING | Fix applied (ca4670bbe), but there is no r5 review; the run was accepted at circling. |
| webster | 4 | gate-test-scope | F1 Whole-plan validate verbs run the plan gate's type load outside a gate slot | B | real behavior bug (unslotted build, which defeats the task's slot cap) | planglyph.Validate's call to defaultTypesLoader came from the original card 9 implementation, and fixers r1-r3 never touched planglyph or the validate verbs. Rounds 1-3 missed it while they focused on test-parallel, refusal-row and gofmt issues. | CONTINUE | Yes: fixed in 97013be91 and CONVERGED in r5, although r5 LOW F3 notes that no test pins the wiring. |

## Webster (loom-bouncer-rounds, reed-multi-window, run-recovery, shuttle-turns)


Warp per-round commits are not reachable (the code repo holds only squash merges; head_sha values in the weft reports are absent from ../loomyard), so origin is judged from the fixer reports and review texts of rounds 1-3.

| perch | round | slug | finding | class | real bug or test/doc | justification | bouncer verdict | resolved next round? |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| webster | 4 | loom-bouncer-rounds | F1 malformed escalation record refuses without a way forward (circling.go:85) | B | operator-message contract (refusal text + spec row), not a behavior bug | The verb-side `refuse("%s", err)` came from batch 08; fixers 1-3 touched bouncer.go, config.go and decision.go only, and round 1 fixed the Bouncer-side twin without sweeping this sibling route. | CIRCLING (parent decided continue) | Yes: fixer 4 added ErrEscalationMalformed; round 5 confirms, though round 5 F1 (MEDIUM) finds further circling refusals forwarding helper errors. |
| webster | 4 | loom-bouncer-rounds | F2 decision add refusals with no retry clause (decision.go:82, :102) | B | operator-message contract, not a behavior bug | The status-read and generic append refusals are batch-13 code; fixer 3 only added the ErrNoDecisionsHeading mapping beside them, and round 3 reviewed decision.go without flagging them. | CIRCLING (parent decided continue) | Partly: way-forward clause added and asserted; round 5 F2 (MEDIUM) notes the two refusals still lack loom spec rows. Round 5 APPROVED. |
| webster | 4 | reed-multi-window | F1 unseen-clear set-hook targeted at bare `=session`, hook never installed (windowsize.go:186) | B | real behavior bug | Fixers 1-2 touched spawn.go, ownwindow.go and test-budget.yaml only; the bug surfaced through the real-tmux publish failure because earlier rounds ran only the fake test, which asserted the same wrong argv. | CONTINUE | Yes: round 5 has no findings, reviewer reran real-tmux tests green; CONVERGED. |
| webster | 4 | reed-multi-window | F2 OpenWindow chained options land on a same-named own window (window.go:68) | B | real behavior bug | The chain-by-exact-name in OpenWindow was original code made unsafe by the branch's own findWindowLocked change; no earlier fixer touched window.go, and it only showed up in the publish-time real-tmux run. | CONTINUE | Yes: fixer 4 used a temporary unique name plus rename; round 5 CONVERGED. |
| webster | 4 | reed-multi-window | F3 pane-gone assertion via display-message cannot detect a missing pane (ownwindow_integration_test.go:563) | B | test-only | The InvalidConfigStillRemovesAndReportsTheKey subtest was original; fixer 1 edited a different subtest (RelaunchOfADeadGroupMemberRejoinsItsWindow), and the false assertion only failed under real tmux at publish. | CONTINUE | Yes: switched to list-panes -s; round 5 CONVERGED. |
| webster | 4 | run-recovery | F1 Go comments join independent clauses at ", so" (battenshed/doc.go:20 and ~30 lines) | C (origin B) | doc/comment style | A pure comment line-break style rule rated BLOCKING; the `, so` lines existed from the start and fixer 2 explicitly left "`so`/`since` clauses as they were", so round 4 re-raises a clause form earlier rounds knowingly skipped. | CIRCLING (parent accepted) | No round-5 review; fixer 4 broke the listed lines (91c021a90) and the parent accepted at circling. |
| webster | 4 | run-recovery | F2 markdown lines join independent clauses (overview.md:477 etc.) | C (origin mostly A) | doc style | Semantic-line-break style issue rated BLOCKING; fixer 3 (1340c2e5d) rebroke the very lines in overview.md (dead-driver and adoption bullets), PATTERN-completion-signal.md, loom-status-spec.md and orch-template-role.md, leaving the `, so` joins in place. | CIRCLING (parent accepted) | No round-5 review; fixer 4 broke the listed lines and the parent accepted. |
| webster | 4 | shuttle-turns | F1 offset-seeding sentence breaks mid-phrase (attach.go:281) | C (origin B) | comment style | Comment-wrap style rated BLOCKING; the sentence's tail was rewritten by the original diff, and fixer 1 reflowed attach.go but treated this legacy-wrapped sentence as exempt. | CIRCLING (parent accepted) | No round-5 review; fixer 4 reflowed it (22f06b1f4), parent accepted. |
| webster | 4 | shuttle-turns | F2 TestAttach_OffsetStartsAtZero doc breaks mid-phrase (attach_test.go:904) | C (origin B) | test comment style | Comment-wrap style in a test doc comment; the tail was rewritten by the original diff and no earlier round listed it. | CIRCLING (parent accepted) | No round-5 review; fixer 4 fixed (8c8e14d00), parent accepted. |
| webster | 4 | shuttle-turns | F3 TestEngine_RetrySemantics doc breaks mid-phrase (treadleengine/engine_test.go:261) | C (origin B) | test comment style | Comment-wrap style; fixer 2 explicitly left the treadle test headers as word-swap legacy wraps, and round 4 reclassified this one as a rewritten clause. | CIRCLING (parent accepted) | No round-5 review; fixer 4 fixed (17b7f87fa), parent accepted. |
| webster | 4 | shuttle-turns | F4 smoke_guardrail_test.go header breaks mid-phrase (:3) | C (origin B, re-raise) | test comment style | Fixer 1 explicitly exempted this header as legacy text with substituted words, and fixer 4 notes round 4 simply re-read the same tail as a rewrite; a re-judgment of an earlier exemption, not a new defect. | CIRCLING (parent accepted) | No round-5 review; fixer 4 reflowed (522a2d1fe), parent accepted. |
| webster | 4 | shuttle-turns | F5 settings.go header breaks mid-phrase (claudeengine/settings.go:1) | C (origin A) | comment style | Fixer 3 (9d0a4fe1c) moved this header's rewritten final clause onto the preceding line to remove a fragment, and round 4 flags the same sentence for still breaking mid-phrase; style issue rated BLOCKING. | CIRCLING (parent accepted) | No round-5 review; fixer 4 fixed (c8157ced4), parent accepted. |

Pattern: the only real behavior bugs (reed-multi-window F1, F2) and the one false test (F3) were present from the start and surfaced only when real tmux ran at publish, so earlier fake-only reviews could not see them.
The run-recovery and shuttle-turns late blockers are all comment/markdown line-break style rated BLOCKING; fixers cleared listed sites one at a time, deliberately exempting some forms, so the next reviewer re-raised the leftovers and the loop CIRCLED; loom-bouncer-rounds is the same enumeration gap for refusal way-forward text.

</details>

