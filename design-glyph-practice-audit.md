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
