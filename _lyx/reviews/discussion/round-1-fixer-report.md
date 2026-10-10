# Round 1 fixer report: discussion

Every finding was checked against the code and fixed in `_lyx/discussion/decision-record.md`, with matching edits in `_lyx/discussion/support-log.md`.
`lyx loom validate-discussion` exits 0 after the edits.

## Fixed

- F1 (BLOCKING): the trace holds.
  `settleUnconverged` marks a budget-cause continue Stuck exempt, `episodeStuckCount` skips it, and a gate-failed round N+1 resolves back to round N's decided Stuck.
  D1 now names which Burler Stucks are exempt (a judged and an unjudged hand-back) and keeps the gate-failed exit counted on the Burler row.
  The Bound is rewritten to name the exempt budget-continue Stuck, to show rather than assert the unjudged ping-pong bound (no verdict means no decision file, so `judgeCall` and `degrade` run), and to bound the gate-failed cycle by the Burler row's counted Stuck.
  Scope item 1, A1, R1, AC1 and the question ledger's Q1 follow, AC2a pins the gate-failed cycle halting, and the support log records the rejected alternative.
  One side claim in the finding is wrong, though the fix doesn't depend on it: no leftover-generation problem survives, because `episodeStuckCount` already ends the Burler row's episode at its segment's Done through `segmentEnders`.
  So D1 now says the row counts the gate-failed rounds of one generation, and it adds the Bouncer's settled-generation clear comment, which also claims a never-resetting Burler episode, to the doc comments it rewrites.
- F2 (BLOCKING): `Wait` calls `pollEventsTick` before `logSessionState`, and only the latter advances the shadow cursor.
  D4 now has the hold decision read turn starts itself, from `PromptOffset` through the tick's batch, ordered by offset, with the guard lifted for the rest of the Wait, and never from the shadow.
  D5 reads its turn start under the same rule.
  The plan-writer note now says this too.
  AC10 gains the same-batch, attached-Wait and shadow-failure rows, and the support log records the rejected shadow sourcing.
- F3 (MEDIUM): the guard is now armed only when the events file holds a turn start before `PromptOffset`.
  The load turns go through the same `sendVerified` as the prompt, so that turn start proves the provider writes them, and a provider that writes none keeps today's behaviour.
  The skip logs at Warn instead of Info.
  D4's Bound states what is still let through (a provider that writes turn starts for the load turns but not for the prompt turn).
  A4, R3 and AC11 follow, and the support log records the rejected second-turn-end bound.
- F4 (MEDIUM): `NewInnerRun` resolves a nil `Now` to `time.Now`, and `Time.Sub` then uses the monotonic difference.
  D3 now requires the wall delta to be taken from stripped readings (`UnixNano` or `Round(0)`), and AC8a pins the stripping at the helper's inputs.
- F5 (MEDIUM): `judgeAgents` judges the api-error check on each run's own `LastActivity` and the fallback on the later of history and episode since.
  D3 now names the stamp each check subtracts from, resets the accumulation on the returned activity stamp, keeps it while activity is unreadable, and states why the subtraction never exceeds any delta's suspended time.
  AC8b covers the api-error and fallback cases.
- F6 (LOW): `looparm.go` and `battencli/wire.go` both skip a held marker and fall through to the file-time reading.
  D5's Bound now says so.
- F7 (NIT): "keeps reporting" became "reports".

## Disputed

None.

## Deferred

None.
