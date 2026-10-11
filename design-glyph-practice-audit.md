# Audit how plans use glyphs in practice, then tune the plan stencil

`glyph-practice-audit` · Note · enhancement, loom, stencil

## Why

`plan-glyph-precision` makes plan cards name the members they change as glyphs, gated by the plan validators, and its committed test proves the glyph chain over a fixture module.
That proves the machinery, not that plan writers use it well.
The operator (2026-10-08) doubts a writer can pick the right glyphs from its own reasoning, and expects the gate to cost Plan rounds and time.
Kick-start packs (`kick-start-pack`, `enclose-kickstart`) build on plans whose glyphs resolve, so they wait until practice shows that.

## Proposal

Audit the first runs after `plan-glyph-precision` lands from their logs and transcripts:

- the share of `Edit:` targets that are member glyphs rather than file paths or self glyphs;
- the Plan rounds and Plan wall time spent on glyph findings (unresolved glyphs, redundancy, caller coverage, the re-sign arrow), against the Plan time of earlier runs;
- `plan:` handles declared, bound at record-batch, and failed;
- members a card edited that its glyphs did not name, and glyphs that named the wrong member, read from the batch diffs;
- how the plan writer arrives at each glyph, read from the Plan-Write transcript: the `lyx quarry` calls it made, the greps and file reads around them, and how many tries a glyph took before it resolved (operator, 2026-10-08: the search may be mostly trial and error, and its cost belongs in the audit);
- test-file paths under `Edit:`: the plan template (`loom-template-plan.md`) allows a plain path for "a test file the card adds tests to", and the rubric flags only non-test Go paths, so every test file goes in as a path (operator, 2026-10-08); count from the batch diffs how many of those paths changed existing members (a test, a fake such as `fakes_test.go`, a helper) that a glyph could have named, and whether new tests could be declared as `plan:` handles like new production members;
- whether implementer and Burler forks use the card's glyphs at execution time (a `lyx quarry` lookup) or fall back to grep and whole-file reads.

## Findings (2026-10-10)

### Method and limits

- Landed runs' plan cards are gone with their worktrees, so each run's final plan is rebuilt from its Plan-Write transcripts under `~/.claude/projects/`: the last `Write` per `_lyx/plan/NN-*.md` card, `Edit:`/`Create:` lists counted; later `Edit` calls on a card are not applied.
- A "Plan round" is a session opened with the Plan-producer prompt (first write plus each respawn after a review finding); Plan-Review lens sessions were not separable, so review-side glyph findings could not be counted.
- "Glyph findings" counts Bash results inside the Plan-Write sessions that carry `glyph-not-found`/`glyph-rejected`/`glyph-ambiguous`/`glyph-malformed`; redundancy, caller-coverage and re-sign findings could not be told apart from stencil and rubric text that quotes the same codes, so they are not reported.
- Plan minutes are the sum of the Plan-Write sessions' first-to-last timestamps (UTC); `shuttle-send` and `gate-test-scope` paused across hours and are left blank.
- The first six runs predate the glyph rules (plan-glyph-precision landed 2026-10-08 11:10) and form the baseline; the last four rows are in flight and read from their transcripts like the rest.
- Not answerable from the data: `plan:` handles bound or failed at record-batch (no trace line records them, and the implementer transcripts do not log it); members a card edited that its glyphs did not name (no archive branches survive, only squash commits); Burler fork behaviour at execution time (forks not separated from the review lens sessions).

### Per run

Targets are `Edit:` plus `Create:` entries; glyph includes `plan:` handles.
Test paths are plain `*_test.go` paths under `Edit:`.

| Run | Targets | Glyph share | Plan rounds | Plan min | Glyph findings (not-found / other) | `plan:` handles | Test paths under Edit |
|---|---|---|---|---|---|---|---|
| batcher-v2 (baseline) | 65 | 0% | 2 | 18 | n/a | 0 | 27 |
| burler-cost (baseline) | 103 | 14% | 5 | 62 | n/a | 14 | 31 |
| shuttle-state (baseline) | 96 | 14% | 2 | 15 | n/a | 13 | 26 |
| lifecycle-waits (baseline) | 106 | 0% | 2 | 15 | n/a | 0 | 38 |
| reed-view (baseline) | 178 | 0% | 3 | 35 | n/a | 0 | 58 |
| review-loop-convergence (baseline) | 87 | 0% | 7 | n/a | n/a | 0 | 31 |
| webster-dead-ends | 312 | 48% | 2 | 37 | 1 / 0 | 21 | 90 |
| review-lens-fan | 78 | 65% | 1 | 11 | 0 / 0 | 14 | 10 |
| binary-change-and-rules | 185 | 56% | 2 | 28 | 0 / 0 | 53 | 38 |
| webster-record-batch | 68 | 65% | 1 | 9 | 1 / 0 | 16 | 14 |
| plan-gate-checks | 50 | 62% | 1 | 10 | 6 / 4 | 16 | 8 |
| shuttle-send | 146 | 65% | 3 | n/a | 2 / 1 | 32 | 38 |
| board-lock-running | 25 | 56% | 1 | 4 | 0 / 0 | 9 | 3 |
| git-robustness | 65 | 69% | 1 | 8 | 0 / 0 | 12 | 8 |
| multi-llm-producer | 117 | 68% | 1 | 13 | 0 / 0 | 62 | 5 |
| gate-test-scope | 226 | 59% | 2 | n/a | 1 / 2 | 51 | 42 |
| bolt-and-store | 126 | 70% | 2 | 30 | 0 / 0 | 50 | 19 |
| daemon-wakeups | 190 | 61% | 2 | 30 | 0 / 0 | 47 | 38 |
| test-suite-weight (in flight) | 368 | 42% | 1 | 28 | 0 / 1 | 98 | 191 |
| driver-context (in flight) | 202 | 62% | 2 | 34 | 1 / 0 | 79 | 20 |
| discussion-multi-llm (in flight) | 160 | 80% | 3 | 37 | 0 / 0 | 32 | 11 |
| publish-recovery (in flight) | 52 | 71% | 2 | 19 | 0 / 0 | 7 | 13 |

`review-loop-convergence` straddled the landing: its seven round-like sessions include non-writer sessions, so its round count overstates.

### Answers

- **Glyph share.**
  The sixteen runs written under the new rules name 1392 of 2370 targets as glyphs (59%), against none to 14% before.
  Of the 978 plain targets, 548 are test-file paths under `Edit:`; almost all the rest are `doc.go`, `template.yaml`, markdown or new whole files.
  Non-test, non-`doc.go` Go source paths remain only in `webster-dead-ends` (about nine, written first after the landing) and `discussion-multi-llm` (three, `spec.go` and `review.go`), so the rubric flag holds.
- **Plan rounds and time.**
  Baseline runs took 2.6 Plan rounds on average (burler-cost outlier at five), the glyph-era runs 1.8; minutes per target fall from about 0.2 to about 0.15.
  Plan time follows plan size (4 minutes for 25 targets, 28 to 37 for 160 to 370), not glyph findings, and no run shows a Plan round spent on a glyph finding alone.
  The operator's expectation that the gate costs rounds and time is not borne out in this sample.
- **Glyph findings.**
  Writer sessions carry zero to two unresolved-glyph results per run (`plan-gate-checks`, the task that built the checks, had six).
  Findings are fixed in one pass, as with `shuttle-send` earlier.
- **`plan:` handles.**
  About 600 handles were declared across the sixteen runs, 7 to 98 per run, so writers use them freely for new members.
  Bind and failure counts at record-batch are not observable; a trace line or board field recording them would make this auditable.
- **How the writer reaches a glyph.**
  It is bulk listing, not trial and error: a Plan-Write session makes 0 to 14 `lyx quarry` calls (typically 6), mostly `lyx quarry glyphs --text <package> | grep -E <names>` per touched package, then one or two batched `lyx quarry resolve` calls over the glyphs it is least sure of.
  Around the quarry calls it still spends roughly half its tool calls on `grep`, `find`, `sed` and `cat` to understand the code, with 20 to 160 tool calls per writer session.
  Some respawn sessions make no quarry call and reuse the prior plan's glyphs.
- **Test paths under `Edit:`.**
  Test paths are 4% to 52% of `Edit:` entries per run (191 of 368 targets in `test-suite-weight`).
  Of 93 distinct test paths in the four squash commits checked (`gate-test-scope`, `bolt-and-store`, `daemon-wakeups`, `multi-llm-producer`), 62 changed existing lines and 29 only added lines; hunk headers name an existing function in 28 of 32 and 35 of 37 files in the two largest, an upper bound since a hunk after an old function shows its name.
  So roughly two thirds of the test edits touch existing members (tests, fakes, helpers) a glyph could have named.
  New tests are not declared as handles in practice: two `plan:...#Test*` handles appear across the four runs, because the template and rubric steer new tests to prose in `Intent`.
- **Execution time.**
  Across 61 implementer sessions (eight runs), `lyx quarry` was called zero times in about 3300 tool calls, against 690 `grep`/`rg` calls.
  Implementers ignore the card's glyphs as a lookup key and search by text.
  A search of `contracts/stencils` finds `quarry` only in the plan template, the prior-plan template and the discussion-review rubric, not in any implementer stencil.

### Conclusion and tuning

Plan writers use glyphs well: the share is steady near 60% with the remainder explained by test paths, `doc.go` and non-Go files, and the gate costs neither rounds nor time that the data can show.
The kick-start packs' precondition ("plans whose glyphs resolve") holds.
The two open weaknesses are test files and execution-time use.

- **Let test members be glyphs.**
  Change `loom-template-plan.md` (line "a test file the card adds tests to") and the rubric so a card names the existing test, fake or helper it changes as a glyph (`internal/x#TestFoo`, `internal/x#fakeClock`), keeping a plain path only for a test file with no member change or a new test file; add a rubric finding for an `Edit` test path whose `Intent` names an existing test function.
  Declare new tests as `plan:` handles (or require them under `Create:`) so kick-start packs can resolve them.
- **Do not add more gate rounds for glyphs.**
  Findings are rare and one-pass; keep the checks and spend no effort tightening the self-check loop.
- **Tell implementers about the card's glyphs.**
  Add a line to the implementer stencil that every `Edit:` glyph resolves with `lyx quarry resolve '<glyph>'` to file, line and signature, and that the card's glyphs replace a grep for the member; re-run this audit's execution-time count afterwards, since zero use is the baseline.
- **Make bind outcomes observable.**
  Log a trace line per `plan:` handle at record-batch (bound, failed, reason), so the next audit can count them.
- **Keep the stencil's quarry advice as is.**
  `glyphs --text <pkg> | grep` then a batched `resolve` is what writers already do and what keeps findings at zero to two; add that recipe to the stencil as the worked example rather than leaving it implicit.

## Follow-up audit

a28e79d9c (2026-10-10) made the implementer stencil resolve the card's target and `Uses:` glyphs with one `lyx quarry resolve` call before any grep.
Once a good number of runs have landed on it, an agent repeats this audit's execution-time count (quarry calls per implementer, grep-for-member calls, whole-file reads) against the zero-use baseline above, and re-checks the plan-side numbers.
Like the first pass, it is a quick read-only survey by one agent, not a loom run.

## Follow-up findings (2026-10-11)

### Method and limits

- **Which implementer sessions qualify.**
  a28e79d9c (2026-10-10 12:12 +0200, 10:12Z) changed `contracts/stencils/webster/webster-body-implementer.md`, which the Master's implementer forks read at the start of each batch.
  A session qualifies when that Read result in its own transcript contains the new "Locate them with one `lyx quarry resolve <glyph>...` call" line; the test is on the transcript, not on the landing date.
  The first such fork started at 10:13Z, so the stencil took effect within about a minute of the commit.
- **Population.**
  The transcripts hold 71 implementer forks (subagent sessions described "Implement batch N") since 2026-10-10 04:00Z.
  51 of them, in 20 runs, read the new stencil: 34 on 2026-10-10 (10:13Z to 18:27Z) and 17 on 2026-10-11 (08:31Z to 12:07Z).
  The other 20 forks, in 7 runs (`git-robustness`, `gate-test-scope`, `multi-llm-producer`, `daemon-wakeups`, `bolt-and-store`, `publish-recovery`, and the first three batches of `test-suite-weight`), read the old stencil and serve as a same-method control.
- **Count method.**
  Per fork, unique `tool_use` ids: Bash calls whose command contains `lyx quarry`; Bash calls with `grep`/`rg` anywhere in the command (all) and with it as a leading command (not after a pipe); Read calls without `offset`; Bash `cat <file>` of a `.go`, `.md` or `.yaml` file.
  The batch's own prompt-file Read and the parent's `Agent` launch line are excluded; `Edit`/`Write` are counted apart.
  "Grep for a member the card names" is approximated by a grep whose pattern holds a Go-style identifier (`func `, `type `, or a CamelCase word); the cards are gone, so the match against the card's own glyphs is not exact.
- **Comparing with the first audit.**
  The first audit's baseline (about 3300 calls, 690 `grep`/`rg` over 61 forks, so 11 greps per fork) cannot be reproduced with this counting: the 20-fork old-stencil control counted here gives 44 greps per fork.
  The first audit evidently counted a narrower set of calls.
  Every comparison below is therefore new forks against the old-stencil control counted the same way; the zero quarry calls of the baseline holds in both (0 in 20 control forks).
- **Plan side.**
  Rebuilt as in the first audit (last `Write` per `_lyx/plan/NN-*.md` card in the run's `ly:<slug>:plan` session, `Edit:` plus `Create:` entries).
  Reproduction check: `test-suite-weight` (368 targets, 42%, 98 handles, 191 test paths) and `gate-test-scope` (226 targets, 59%) match the first audit exactly.
- **Limits.**
  Forks inherit the Master's context, so quarry use may follow the Master's habits as much as the stencil.
  Plan "rounds" are not comparable with the first audit's: respawns here are Burler-fix sessions, not new `:plan` sessions, and I count `:plan` sessions plus fixer sessions that touched a plan card as a proxy.
  Plan minutes are the `:plan` session span only.
  Glyph findings are counted from `lyx loom validate-plan` results only: counting every Bash result that holds a `glyph-*` code, as the first audit did, over-counts because `cat`/`grep` of `internal/planglyph/*.go`, the spec and the stencils quotes the codes (it gave 6 / 11 for `webster-stability` against 0 / 0).
  Still not observable: `plan:` bind outcomes at record-batch, members edited that no glyph named, and which implementer forks were Burler fixers.

### Per run

Implementer columns count only forks on the new stencil.
Calls exclude `Edit`/`Write`.
`Quarry` is forks that called it / calls.
`Whole reads` are Read without `offset`.
Plan columns: targets (glyph share), `:plan` sessions, their minutes, glyph findings from `validate-plan` (not-found / other), fixer sessions that touched a card, `plan:` handles, and test paths under `Edit:` / `#Test*` glyphs under `Edit:`.

| Run | Forks | Quarry | Calls | grep/rg | Whole reads | cat file | Targets | Plan sess | Plan min | Findings | Fixers | `plan:` | Test paths / test glyphs |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| burler-multi-llm | 2 | 0 / 0 | 231 | 75 | 29 | 14 | 127 (57%) | 1 | 17 | 0/0 | 1 | 13 | 30 / 0 |
| cli-surface | 3 | 0 / 0 | 242 | 103 | 1 | 27 | 197 (79%) | 1 | 19 | 0/0 | 0 | 22 | 26 / 7 |
| discussion-multi-llm | 3 | 0 / 0 | 157 | 74 | 3 | 22 | 130 (82%) | 2 | 30 | 0/0 | 4 | 31 | 7 / 21 |
| driver-context | 4 | 2 / 3 | 437 | 207 | 32 | 26 | 202 (62%) | 1 | 24 | 0/0 | 4 | 79 | 20 / 0 |
| fixer-critical-of-findings | 1 | 0 / 0 | 25 | 5 | 6 | 0 | 13 (46%) | 1 | 3 | 0/0 | 1 | 0 | 0 / 2 |
| gate-test-hygiene | 2 | 2 / 4 | 132 | 63 | 8 | 11 | 83 (65%) | 1 | 14 | 1/0 | 2 | 19 | 2 / 13 |
| loop-small-bugs | 1 | 1 / 1 | 88 | 30 | 2 | 0 | 47 (91%) | 1 | 10 | 0/0 | 2 | 9 | 0 / 12 |
| pane-inject-safety | 3 | 1 / 1 | 303 | 133 | 15 | 10 | 175 (76%) | 1 | 13 | 0/0 | 1 | 49 | 14 / 19 |
| per-role-models | 2 | 1 / 1 | 131 | 48 | 24 | 2 | 58 (93%) | 1 | 13 | 0/0 | 4 | 11 | 0 / 18 |
| plan-gaps | 1 | 0 / 0 | 79 | 38 | 1 | 8 | 59 (68%) | 1 | 11 | 0/0 | 2 | 38 | 4 / 0 |
| quick-recipe | 3 | 1 / 1 | 338 | 158 | 35 | 22 | 216 (63%) | 1 | 21 | 0/0 | 2 | 63 | 39 / 0 |
| reed-hygiene | 3 | 1 / 1 | 156 | 73 | 5 | 6 | 58 (53%) | 1 | 12 | 0/0 | 2 | 15 | 10 / 0 |
| reed-tmux-conf | 1 | 0 / 0 | 76 | 37 | 0 | 4 | 51 (84%) | 1 | 17 | 0/0 | 1 | 15 | 0 / 10 |
| run-time-by-role | 1 | 0 / 0 | 53 | 16 | 0 | 7 | 52 (71%) | 1 | 7 | 1/0 | 2 | 22 | 6 / 0 |
| small-fixes-batch | 2 | 0 / 0 | 145 | 58 | 9 | 7 | 94 (77%) | 1 | 12 | 0/0 | 2 | 10 | 0 / 28 |
| test-suite-weight | 6 | 1 / 1 | 359 | 102 | 105 | 24 | 368 (42%) | 1 | 28 | 0/1 | 3 | 98 | 191 / 0 |
| twill-discusser | 2 | 0 / 0 | 107 | 45 | 6 | 10 | 60 (77%) | 1 | 12 | 0/0 | 2 | 22 | 3 / 10 |
| webster-audit-gaps | 2 | 0 / 0 | 120 | 60 | 1 | 3 | 54 (59%) | 1 | 12 | 0/0 | 2 | 7 | 12 / 0 |
| webster-replan-state | 6 | 5 / 6 | 528 | 253 | 16 | 19 | 201 (54%) | 1 | 19 | 0/0 | 3 | 20 | 57 / 0 |
| webster-stability | 3 | 1 / 1 | 209 | 76 | 15 | 11 | 114 (55%) | 1 | 16 | 0/0 | 2 | 12 | 28 / 0 |
| Total (20 runs) | 51 | 16 / 20 | 3916 | 1654 | 313 | 233 | 2359 (64%) | 21 | 310 | 2 / 1 | 42 | 555 | 449 / 140 |
| Control, old stencil (20 forks, 7 runs) | 20 | 0 / 0 | 1593 | 890 | 61 | 142 | 763 (64%) | 6 | 73 (ex `gate-test-scope`, paused) | n/a | 17 | 222 | 123 / 1 |

### Answers

- **Which runs qualify.**
  Twenty runs, 51 forks: `discussion-multi-llm`, `test-suite-weight` (six of its nine forks), `driver-context`, `webster-replan-state`, `cli-surface`, `quick-recipe`, `reed-hygiene`, `webster-audit-gaps`, `loop-small-bugs`, `webster-stability` on 2026-10-10, and `small-fixes-batch`, `gate-test-hygiene`, `per-role-models`, `fixer-critical-of-findings`, `twill-discusser`, `plan-gaps`, `reed-tmux-conf`, `pane-inject-safety`, `run-time-by-role`, `burler-multi-llm` on 2026-10-11.
  That is a meaningful sample, though the plans of the first ten were written before the stencil changed (the stencil is read at execution, so that does not matter for this count).
- **Quarry calls per implementer.**
  Baseline zero; now 20 `lyx quarry resolve` calls in 51 forks, 16 forks (31%) with at least one, 0.4 per fork.
  Nothing else of quarry is called (no `glyphs`, no `find`).
  Use fell from 12 of 34 forks on 2026-10-10 to 4 of 17 on 2026-10-11, so it is not growing.
  Of the 16 users, 13 made the call as their first or second action, as the stencil says; 3 called it late (after 23, 88 and 123 calls, behind a grep) or incidentally.
  Twelve of the 20 calls chain a `grep` onto the same command line; the other eight are a bare batch of 5 to 15 glyphs, which is the intended form.
- **Does the glyph being present decide it?**
  No.
  The cards held glyphs in 49 of the 51 forks (on average 37 distinct glyphs in view), and 33 of those never called quarry, so most forks skip the instruction while having glyphs to resolve.
  Forks that used quarry were not smaller or simpler (average 174 calls with edits, against 142 for those that did not).
- **Grep for members.**
  Per fork, `grep`/`rg` calls fell from 44.5 (control) to 32.4, and the grep share of non-edit calls from 56% to 42%.
  Identifier-pattern greps fell from 23.8 to 19.6 per fork.
  Forks that called quarry grepped as much as the rest (38.5 against 29.7 per fork) because they were larger and chained greps after the resolve, so the fall is not shown to come from quarry use; the task mix also differs between the two groups.
  Calls that read spans with `sed -n` stayed at 20 per fork (24 in the control), so forks still read by line range they found with `grep -n`.
- **Whole-file reads.**
  Read without `offset` rose from 3.0 to 6.1 per fork (4.6 without `test-suite-weight`, whose six forks made 105 of the 313, 101 of them on `.go` files), and Bash `cat <file>` fell from 7.1 to 4.6 per fork.
  Whole-file reading did not fall.
- **Plan-side numbers (same 20 runs).**
  Glyph share is 64% of 2359 targets (1510), the same as the six control plans (64% of 763), against 59% in the first audit's sixteen runs.
  Per-run shares run from 42% (`test-suite-weight`, 191 test paths) to 93%, and the three highest are the three runs with test members as glyphs.
  Plan time is 15.5 minutes per run and 0.13 minutes per target, equal to the control (0.14), and one `:plan` session per run except `discussion-multi-llm` (two).
  Fixer sessions that touched a plan card were 42 over 20 runs (2.1 per run), against 17 over 6 control runs (2.8); this is a proxy for plan rounds and gives no sign of a rise.
  Glyph findings in the Plan-Write sessions' `validate-plan` runs were three in 20 runs (`gate-test-hygiene` and `run-time-by-role` not-found, `test-suite-weight` ambiguous),; redundancy, caller-coverage and re-sign findings did not show up in those results beyond three loose matches.
  `plan:` handles: 555 over 20 runs (0 to 98 per run).
  Writers make about 7 quarry calls in 85 tool calls per `:plan` session, as before.
- **Does the test-member recommendation show up?**
  Partly, and without a template change (`loom-template-plan.md` still allows a plain test-file path, and a rubric line was not added).
  Test paths under `Edit:` are 449 of 1744 entries (26%, the control's 25%), but `#Test*` glyphs under `Edit:` rose from 1 in the control to 140.
  On 2026-10-10 only three of ten runs have them (`discussion-multi-llm` 21, `loop-small-bugs` 12, `cli-surface` 7; 40 glyphs against 390 paths), and in the 2026-10-11 runs the split is 59 paths to 100 test glyphs; seven of ten runs that day (`small-fixes-batch`, `per-role-models`, `reed-tmux-conf`, `gate-test-hygiene`, `twill-discusser`, `pane-inject-safety`, `fixer-critical-of-findings`) name tests as glyphs, while `burler-multi-llm` (30 paths, 0 glyphs), `plan-gaps` and `run-time-by-role` still use paths.
  Test fakes and helpers cannot be told from the names, so the glyph count is a floor.
  New tests as `plan:...#Test*` handles: 24 across 8 runs (`discussion-multi-llm` 8, `gate-test-hygiene` 5, `reed-tmux-conf` 4), against 2 in the first audit.
  Non-test, non-`doc.go` Go source paths under `Edit:` remain 12 across seven runs (4 in `gate-test-hygiene`, 3 in `reed-hygiene`); the rubric flag still holds.

### Conclusion and tuning

The stencil line moved implementer behaviour a little and not enough: quarry went from none to a third of forks, one call each, and the majority of forks with glyphs in front of them still search by text and read whole files; use did not rise over the second day.
Grep volume per fork is lower than in the control, but that cannot be tied to quarry, and whole-file reads are up.
The plan side is steady: 64% glyphs, no extra rounds or minutes, glyph findings rare, and writers now name existing tests as glyphs on their own.

- **Make the resolve a step, not advice.**
  The line sits inside step 2, between the grammar pointer and the gate list, and 31% of forks follow it.
  Make it a numbered step of its own ("Before reading or editing any target, run `lyx quarry resolve` with every glyph from the card's `**Edit:**` and `**Uses:**` lists, in one call"), and name the output fields to use (`file`, `start`, `end`) and the follow-up (`sed -n 'START,ENDp' FILE` or a Read with `offset` and `limit`).
  The 13 forks that did it first used it as intended, so the mechanism works when it happens.
- **Say what not to do next.**
  Add "do not `cat` or Read the whole file of a resolved member, and do not grep for it" to that step; whole-file Read per fork doubled and 10 of 20 quarry calls were chained to a grep anyway.
- **Resolve once per batch.**
  A fork works a batch of cards but the line says "the card's"; word it as "every card of the batch" so one call covers the batch and the few late or repeated calls disappear.
- **Skip the `plan:` handles in the call.**
  The stencil already says they do not resolve; list only the non-`plan:` glyphs in the example so the call does not return not-found noise.
- **Next measurement.**
  Re-run this count once the step wording is changed and the stencil is in the binary; compare quarry forks (target above 80% where the card has glyphs), greps and whole-file reads per fork against this follow-up rather than the first audit's baseline.
- **Plan side, unchanged from the first audit with one update.**
  The test-member change to `loom-template-plan.md` and the rubric is now partly done by the writers; codify it so `burler-multi-llm`-style plans (30 test paths, 0 glyphs) follow, and keep the rule that a new test is a `plan:` handle.
  A trace line per `plan:` handle at record-batch (bound, failed, reason) is still open.
