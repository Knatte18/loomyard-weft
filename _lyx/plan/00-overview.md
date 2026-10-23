---
format: 5
approved: true
language: go
---

# Plan: loom-tail-review — rebuild the run tail after the code is written

Rebuild loom's and darn's tails as the decision record decides.
Loom runs Webster, then an unlocked Merge-In with a merge fixer that also aligns, then the Code-Perch, Describe, a PR-Review stub behind a flag and one locked Finalize block; darn runs its writer, Darn-Review, Describe, its own PR-Review stub and the same Finalize.
Each recipe fixes its own merge and test failures inside these rows, the parent lock covers only Finalize, the landing halts go to the orch through every driver, and the PR rows, the Merge-Review perch, the approval verbs and every route that sent a verify failure to another row go.
The cards first add the building blocks (fabric's merge of a given SHA, the fixer's report, session, caps, attempt record and loop), then remove the old failure routing, then swap the recipes, then rebuild Merge-In and Finalize, then mark and hand back the landing halts, then delete the PR surface and migrate the config, and last land the docs and PATTERN lines.

## Card Index

1 — fabric-merge-in-at — fabric merges a given parent SHA through MergeInAt
2 — resolve-at-tip — the conflict-resolution seam carries the merged tip
3 — test-file-patterns — per-language test-file patterns beside sourceExtensions
4 — reroute-cap-prefix — name the reroute-cap reason prefix as a constant
5 — landing-fixer-config — landing.yaml's fixer, timeout, attempt and line-cap keys
6 — fixer-report — the fixer report spec and its one parser
7 — fixer-session-table — the merge fixer's stencil and session table
8 — fabric-line-change — fabric reads a commit range's line count, paths and added lines
9 — fix-cap — judge a fixer change against the size and kind caps
10 — fixer-attempts — the per-row-entry fixer attempt record
11 — fixer-run — the merge fixer loop Merge-In and Finalize share
12 — fixer-wiring — fill the fixer's seams in loom's landing assembly
13 — darn-write-no-reroute — Darn-Write loses its failure reroute and rejection read
14 — settled-guard-no-verify-exception — the settled-segment guard drops its verify-failure exception
15 — round-gate-drops-failure-record — the round gate and the review rubric stop reading the failure record
16 — driver-drops-verify-row — the Go driver drops its verify-halt row and seam
17 — arbiter-drops-verify-rule — arbiterstop drops its verify-halt rule
18 — batten-drops-publish-retry — batten drops its publish retry
19 — landing-drops-failure-routes — the failure record loses its target and route classifier
20 — pr-review-stub — the PR-Review stub engine and its flags
21 — code-perch-keys — webster_* perch keys become code_* with a migration
22 — recipe-tails — both recipes take their new tails and the Code-Review ring
23 — merge-review-retired — the Merge-Review seams, rubric and keys go
24 — merge-record-aligned-tip — the merge record's reviewed tip becomes the aligned tip
25 — merge-in-unlocked — Merge-In merges the upstream tip with no parent lock
26 — merge-in-fixer — Merge-In runs the merge fixer in merge-in mode
27 — finalize-lock-first — Finalize takes the lock first, fast-forwards and releases on every exit
28 — finalize-merge-judgement — Finalize merges the parent and reroutes a conflict or large merge
29 — finalize-gate-and-fixer — Finalize's landing gate, finalize-mode fixer and caps
30 — finalize-recheck-and-land — Finalize re-checks its lock and lands with the task trailer
31 — review-input-reentry — the Bouncer re-entry seam feeds a newer review input
32 — merged-tip-merge-base — every task-diff reader takes the merged tip's merge base
33 — landing-gate-at-fix-rows — the fix rows run the landing gate
34 — hand-back-mark — the hand-back predicate and its envelope and status mark
35 — driver-hands-back-landing-halts — the Go driver hands back the landing halts
36 — arbiter-refuses-landing-halts — the arbiter refuses the landing halts
37 — batten-restep-skips-landing-halts — batten's binary re-step skips the landing halts
38 — driver-stencils-hand-back — the LLM driver stencils skip marked halts
39 — reroute-cap-way-forward — the reroute-cap clause names Finalize's fixer causes
40 — idle-release-keeps-finalize — idle release keeps the lock of a running Finalize
41 — batten-drops-pr-gate-lock-arms — batten's PR-Gate lock arms and the loop hand-off go
42 — landing-drops-pr-producers — Publish, PR-Align, PR-Gate and the pull-request code go
43 — drop-approve-reject-verbs — lyx loom approve and reject go
44 — drop-pr-rework — PR-Rework, its gate, stencil and rework key go
45 — drop-pr-gate-alignment-point — the alignment pr-gate point and alignment_pr_approve go
46 — landing-failure-record — the Publish failure record becomes the landing failure record
47 — deploy-branches — require_pr_to_base becomes deploy_branches
48 — drop-publish-gate-kind — the publish gate kind goes with a gate migration
49 — run-settings-migration-warn — run-settings migration lines log at Warn
50 — report-readers — the reports read old and new runs
51 — drop-retired-row-names — delete the retired row-name constants
52 — docs-and-pattern — PATTERN lines, background and cross-cutting docs

## Shared Decisions

### Decision: fabric vocabulary at the merge seam

- **Decision:** only `internal/fabricengine` names the code side in an identifier (`MergeInAt(source, warpSHA)`); `internal/mergeresolve` and `internal/landingshed`, outside the vocabulary owner set, call the same value `tip`.
- **Rationale:** `PATTERN-fabric-vocabulary` bars a fabric side token in those packages' identifiers, strings and comments.
- **Applies to:** cards 1, 2, 25

### Decision: one fixer loop, two modes

- **Decision:** Merge-In and Finalize share one loop (card 11) over one session table (card 7), one report parser (card 6), one attempt record keyed by row entry (cards 10, 12) and one cap judge (card 9); each row decides only when to spawn, which cap applies and where a spent cap or an over-cap change goes.
- **Rationale:** D4 makes the fixer one session shared by both rows; one loop keeps their halts and attempt accounting identical.
- **Applies to:** cards 6 to 12, 26, 29

### Decision: the fixer turn bound

- **Decision:** a turn is charged into the attempt record when it starts, so a crash, a timeout or a resume never grants a fresh turn, and an `escalate` verdict or a failed spawn takes its charge back; a turn that returns past `fixer_timeout_min` is the timeout end, and the session table's timeout, `fixer_timeout_min` times the turns the entry has left, ends a turn that hangs and never returns.
- **Rationale:** the seat engine bounds a session run, not one turn, so the gate closure holds each returning turn to `fixer_timeout_min` and the session bound caps a hung one.
- **Applies to:** card 11

### Decision: the hand-back predicate lives in landingshed

- **Decision:** `landingshed.HandsBack(producer, reason)` is the one predicate; shedverbs is told it as `Spec.HandBack`, the Go driver as `Deps.HandBack`, batten as `InnerRunDeps.HandBack`, and arbiterstop calls it directly.
- **Rationale:** the predicate reads Merge-In's and Finalize's own row names and `LandingStopPrefix`, which landingshed declares; shedverbs, sheddriver and battenshed stay recipe-generic.
- **Applies to:** cards 34 to 37

### Decision: retired names stay readable for old runs

- **Decision:** the report declares the retired row names as its own literals and reads an old run's `webster` review directory as Code-Review; old `reviewed_tip`, `reject_review` and `publish-failure.yaml` records read as absent.
- **Rationale:** `lyx report convergence` must run over runs recorded before and after the change, and `PATTERN-tolerant-records` reads a dropped field as absent.
- **Applies to:** cards 24, 31, 46, 50

## PATTERN changes

- `PATTERN-parent-lock` — changes: Moving, merging into or landing on a local parent branch: only in Finalize, under its `internal/parentlock` lock.
- `PATTERN-recipe-owns-failures` — adds: A merge or test failure at Merge-In or Finalize: its fixer, a Finalize reroute or a halt to the orch.
- `PATTERN-owner-machine` — changes: Starting, resuming or landing a run or acting on wake: only where no other machine owns its entry. — [background](pattern/PATTERN-owner-machine.md)
- `PATTERN-pause-holds-prompts` — changes: Typing into a covered session: shuttle refuses by slug; caller holds the line or claims the session. — [background](pattern/PATTERN-pause-holds-prompts.md)
- `PATTERN-model-choice-mapping` — changes: Mapping a model-spec onto a spawn: only via `shuttleengine.NewModelChoice`. (test) — [background](pattern/PATTERN-model-choice-mapping.md)
- `PATTERN-failure-stop-visible` — changes: An `--until-stop` failure as sole holder: status is non-running before the waiter returns. (test) — [background](pattern/PATTERN-failure-stop-visible.md)
