# Support log: small-fixes-oct11

## Interview

The session ran autonomously, so the interview was conducted against the codebase and recent history; every pick is in the question ledger.

1. Board read: one task entry with five sub-items; no other board entry plans work in `tools/tokencount`, the loom step help, the plan rubric or batten notices.
   `shuttle-startup-ready` touches batten seat holds, not notices.
2. Crash gap: `workingIntervals` opens an interval after every point that does not end the turn, and the last point of a transcript opens none.
   So the leaked gap needs a later line in the same transcript, which a `--resume` (shuttle's crash recovery uses `--resume <id>`) supplies.
   Real transcripts show `stop_reason: null` on intermediate content blocks mid-turn, ruling out "null ends the turn".
   Settled on D1: the interval into a turn-start line is idle.
   Review round 1 brought transcript evidence of stalls with no prompt (a suspended host: a `tool_use` line followed 32 hours later by its `tool_result`) and of compaction summaries that the turn-start test would have made idle.
   D1 gained a 30-minute stall cap and excluded the compaction summary line from turn starts.
3. Slug reuse: time is already clipped to the span, whose floor is the next-older tag's date, so the leak is mainly tokens, forks, Merriam starts, the base SHA and the run window the orch charge reads, plus time for runs without a shed record.
   Settled on D2: a lower bound per run from the previous run's archive tag, judging each transcript by its first line.
4. Loom step: the help claims step bootstraps exactly as start does, while `resolveRunID` refuses every generic verb without a seed.
   With a seed, step does call `seedAndCommitBootstrap`, so the accurate text is "needs a seed, then runs start's bootstrap block".
   Settled on D3 and D4, extending the refusal's remedy for `run` and `step`.
5. Rubric: the "Proposed fixes" section (added 2026-10-08) already requires every validation check to admit the fix; issue #542 (2026-10-11) shows that generic pointer was not enough.
   Settled on D5: name the two redundancy checks, mirroring the plan template's pointer.
6. Batten notices: commit `17b09e97f` replaced `; child state X` with `haltText` for state-changed notices; `conditionText` returns the finding's own text for quiet and api-error and a fixed string for driver-dead, so those lost the state.
   Settled on D6: prefix `haltText`, as the driver-busy notice does.

## Rejected alternatives

- D1, a cap alone, or one near ten minutes: a cap near ten minutes cuts a legitimate foreground tool call, and a cap alone leaves a resume's gap under the cap counted, which the turn-start rule drops at any length.
- D1, count an over-cap interval up to the cap: credits every stall with the cap's full length of working time.
- D1, count compaction as idle: compaction is model work with a token cost, triggered by lyx itself through distill and refresh cycles.
- D1, treat a null `stop_reason` as a turn end: intermediate content blocks carry null mid-turn, so working time would collapse.
- D2, bound by the seed's `started_at`: absent for runs without a status or seed, and in flight, and circular with the transcript window `useShedRecord` falls back on.
- D2, an upper bound at the span's end: no newer run of the slug can share the project directory with the counted run, so it fixes nothing about reuse and would drop real cost after the last history entry.
- D2, filter per line: lyx never carries a session across runs, and per-line filtering would still leave old sessions in the session count, forks and Merriam starts.
- D2, judge a transcript by its last line: an older run's driver session can write after that run's final tagged commit and would be counted whole.
- D3, make step seed when absent, matching the old help: changes behaviour the batch decision "loom's run and step do not auto-seed" settled, and only `start` owns the commit-before-precondition ordering.
- D4, read the recorded parent branch into the refusal: adds an origin-record read and lock to a check kept to seed and run-list reads.
- D4, keep the disagreeing-seed refusal's generic "keep the existing seed and drive it" and record it as an open risk: on this path driving the existing seed is what was just refused, so an agent following the message loops.
- D4, name the driverless seed for every generic verb: `status`, `pause` and `goto` gain nothing from a driverless seed.
- D5, restate the rule ("a file target and a member glyph in the same file are mutually exclusive") in the rubric: paraphrases the plan spec, against `PATTERN-producer-pointer-rule`.
- D5, point reviewers at `lyx loom validate-plan` as a pre-check: needs a trial edit of the plan, which a reviewer does not make.
- D6, append `; child state running` after the condition: the old form; `haltText` also names the row, which is what the orch acts on.

## Review rounds

_No rounds yet._

## Question ledger

1. Resolved (auto-pick): crash gap rule — turn-start rule plus a 30-minute stall cap that drops an over-long interval whole; compaction stays working time (D1).
2. Resolved (auto-pick): slug reuse bound — previous run's archive tag date, lower bound only, per transcript by first line (D2).
3. Resolved (auto-pick): report the left-out transcripts — one per-run line with count and bound date (D2).
4. Resolved (auto-pick): which help texts change — `step` and `run` (D3).
5. Resolved (auto-pick): which verbs' refusals name the driverless seed — `run` and `step` only (D4).
6. Resolved (auto-pick): the parent branch in the seed command — placeholder `<parent-branch>` (D3, D4).
7. Resolved (auto-pick): rubric fix shape — name the two checks by pointer, no restatement, no validate-plan pre-check (D5).
8. Resolved (auto-pick): notice wording — `haltText` prefix and drop `while the child is running` (D6).
9. Resolved (transcript evidence): whether an abandoned turn's gap can end at a line other than a prompt — it can, at assistant, `tool_result` and interrupt-marker lines; the stall cap covers those (D1).
10. Resolved (auto-pick): a transcript with no timestamped user or assistant line under a bound — left out (D2).
11. Resolved (auto-pick): the driverless route for `run` — a first `lyx loom step` seeds the status file, and `run`'s status-file refusal names it beside `lyx loom start` (D3, D4).
12. Resolved (auto-pick): a wrong hand-typed parent branch — the help names where the recorded parent is read, and the bootstrap's disagreeing-seed refusal names `rm` of the uncommitted seed and a re-seed with the recorded parent (D3, D4).
