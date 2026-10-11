# Support log: small-fixes-oct11

## Interview

The session ran autonomously, so the interview was conducted against the codebase and recent history; every pick is in the question ledger.

1. Board read: one task entry with five sub-items; no other board entry plans work in `tools/tokencount`, the loom step help, the plan rubric or batten notices.
   `shuttle-startup-ready` touches batten seat holds, not notices.
2. Crash gap: `workingIntervals` opens an interval after every point that does not end the turn, and the last point of a transcript opens none.
   So the leaked gap needs a later line in the same transcript, which a `--resume` (shuttle's crash recovery uses `--resume <id>`) supplies.
   Real transcripts show `stop_reason: null` on intermediate content blocks mid-turn, ruling out "null ends the turn".
   Settled on D1: the interval into a turn-start line is idle.
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

- D1, cap a working interval at a fixed length: cuts a legitimate long foreground tool call (up to ten minutes) or a long thought, and needs an arbitrary constant; the structural rule removes exactly the crash gap.
- D1, treat a null `stop_reason` as a turn end: intermediate content blocks carry null mid-turn, so working time would collapse.
- D2, bound by the seed's `started_at`: absent for runs without a status or seed, and in flight, and circular with the transcript window `useShedRecord` falls back on.
- D2, an upper bound at the span's end: no newer run of the slug can share the project directory with the counted run, so it fixes nothing about reuse and would drop real cost after the last history entry.
- D2, filter per line: lyx never carries a session across runs, and per-line filtering would still leave old sessions in the session count, forks and Merriam starts.
- D2, judge a transcript by its last line: an older run's driver session can write after that run's final tagged commit and would be counted whole.
- D3, make step seed when absent, matching the old help: changes behaviour the batch decision "loom's run and step do not auto-seed" settled, and only `start` owns the commit-before-precondition ordering.
- D4, read the recorded parent branch into the refusal: adds an origin-record read and lock to a check kept to seed and run-list reads.
- D4, name the driverless seed for every generic verb: `status`, `pause` and `goto` gain nothing from a driverless seed.
- D5, restate the rule ("a file target and a member glyph in the same file are mutually exclusive") in the rubric: paraphrases the plan spec, against `PATTERN-producer-pointer-rule`.
- D5, point reviewers at `lyx loom validate-plan` as a pre-check: needs a trial edit of the plan, which a reviewer does not make.
- D6, append `; child state running` after the condition: the old form; `haltText` also names the row, which is what the orch acts on.

## Review rounds

_No rounds yet._

## Question ledger

1. Resolved (auto-pick): crash gap rule — turn-start rule over a time cap (D1).
2. Resolved (auto-pick): slug reuse bound — previous run's archive tag date, lower bound only, per transcript by first line (D2).
3. Resolved (auto-pick): report the left-out transcripts — one per-run line with count and bound date (D2).
4. Resolved (auto-pick): which help texts change — `step` and `run` (D3).
5. Resolved (auto-pick): which verbs' refusals name the driverless seed — `run` and `step` only (D4).
6. Resolved (auto-pick): the parent branch in the seed command — placeholder `<parent-branch>` (D3, D4).
7. Resolved (auto-pick): rubric fix shape — name the two checks by pointer, no restatement, no validate-plan pre-check (D5).
8. Resolved (auto-pick): notice wording — `haltText` prefix and drop `while the child is running` (D6).
9. Open: whether Claude Code writes a synthetic interrupt result at resume time after a crash, which would leave D1's gap counted; recorded under Open risks.
