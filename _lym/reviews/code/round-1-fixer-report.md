# Fixer report — round 1

Gate: `lym gate test ./internal/shuttleengine/claudeengine` passed after the fixes and again after the self-check fixes.

## Changes

- **F1** (fixed as proposed): `TestEngineConstructorsSetTheTranscriptWait` in `batchingnudge_test.go`.
  Rows `New` and `NewFromConfig` assert `tick`, `attempts` and a non-nil `sleep`.
- **F2** (fixed as proposed): four rows after `LateCallPastTheNudgeCapIsSilentWithoutWaiting` in `TestBatchingNudge`:
  `NoStateFileIsSilentWithoutWaitingForALateCall`, `SubagentPastItsOwnCapIsSilentWithoutWaitingForALateCall`,
  `SubagentEditReachingTheTranscriptDuringTheWaitFires` and `SubagentWithNoReadingYetIsSilentWithoutWaiting`.
- **F3** (fixed as proposed): table field `lateAfter`, the sleep closure that grows the transcript with sidechain messages before that wait,
  and the row `EditReachingTheTranscriptAfterSeveralRereadsFires`.
- **F4** (fixed with own wording): `doc.go` and the `awaitCallInTranscript` doc state the one-call silence.
  The proposal said the reading is the previous message's without condition; the sentence now says so only while the call's message is not in the transcript yet.
  The `BatchingNudge` doc states it too.
- **F5** (fixed as proposed, then split): the comment above the wait constants names no figure.
  The proposed single line passed the 120-character comment limit, so it is two lines, and the next line's pronoun became "The wait".
- Self-check fixes: the `late` field comment names `lateAfter`, and the `lateAfter` comment says such a row is a main-session row.

Commits: `dcf93492` (F4, F5), `7de5daf5` (F1, F2, F3), and one for the self-check fixes.
F1, F2 and F3 share one file and one commit, since the three change the same table and closure.

## Self-check

- fix-code-correctness, LOW: F4 sentences state the common case as the rule. Fixed with the condition "While the call's message is not in the transcript yet".
- fix-proposal, MEDIUM: same finding as above (merged). Fixed.
- fix-proposal, LOW: F5 line over 120 characters and a dangling "It". Fixed.
- fix-consistency, LOW: the same dangling "It" (merged). Fixed.
- fix-consistency, LOW: stale `late` comment. Fixed.
- fix-consistency, LOW: the `BatchingNudge` doc lacked the F4 fact. Fixed.
- fix-proposal, LOW: `EditReachingTheTranscriptAfterSeveralRereadsFires` does not pin the size gate against a reread on every attempt. Disputed, see below.
- fix-proposal, LOW: `lateAfter` is safe only on main-chain rows. Fixed by a comment on the field; no row combines the two.
- fix-affected-set: no findings.

## Disputed

- fix-proposal, LOW, the size gate: every fix worse.
  A gate that rereads on every attempt returns the same results as one that rereads on a size change, since an unchanged file yields the same tail.
  No row can tell them apart by outcome, so a test would assert the number of reads and so the internals.
  The row pins the behavior F3 asked for: a gate that never rereads fails it.

## Deferred

None.
