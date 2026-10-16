# Burler lens fan in production: per-segment lenses for Discussion and Plan review

`fan-prompt-tuning` · Task · enhancement, burler, stencil, loom

Depends on: [`webster-fork-stencil-fixes`](design-webster-fork-stencil-fixes.md)

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
