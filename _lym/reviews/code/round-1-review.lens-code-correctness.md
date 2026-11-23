```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each function and branch the diff adds or changes with concrete inputs.

**Per function or branch: the case I ran, and the result**

- `creelengine.Store.AwaitsReply`:
  - Empty strand: refused with a way forward. Correct.
  - An unread question from alpha to orch, with orch a strand: counts. Correct.
  - A question the holder read, then the holder was removed: `releaseGoneHolder` settles it `holder-gone`, so `Unanswered` is set and it stops counting. Correct.
  - An unread question whose holder was removed: it is returned, `EndedUnreadBy` is set, and it stops counting. Correct.
  - A question to a program: `isStrandName(Holder)` is false, so it does not count. Correct.
  - A reply alpha has not read: `ReplyTo` points at alpha's message and status is unread, so it counts. Correct.
  - Mail with `ReplyTo` 0: the map lookup gives nil, so it does not count. Correct.
  - A reply with status Delivered or Undelivered: only an EXTERNAL holder or a non-strand step folds that way, so a strand asker is unaffected. Correct.
  - A question from an earlier session that had the same strand name: counts. Incorrect, see F2.
- `server.go` `VerbAwaitsReply` dispatch: a store error is returned and the answer is set otherwise. Correct.
- `GoClient.AwaitsReply` and `AwaitsReplyProbe`: a client construction error and a call error both reach the caller. Correct.
- `Run.awaitsReply`:
  - Nil probe or empty strand: false. Correct.
  - Probe error: warns once and answers false. Correct.
- `Run.holdForReply`: true starts a fresh hold with the probe time set to now. Correct.
- `Run.releaseReplyHold`:
  - Less than 5 s since the last probe: no-op. Correct.
  - Awaiting: the quiet count resets. Correct.
  - Two quiet probes on an ungated run: finalizes done. Correct.
  - Gated run under a pause: sets `boundaryHeld`, and the pause-lift branch probes again. Correct.
  - Gated run uncovered: `boundaryIdle` then `handleGatedBoundary`. Correct.
  - A reply that was read but whose woken turn writes no turn start: releases while the agent works. Incorrect, see F1.
- `Wait` clear at the top (`eventsRead || turnStartSeen`):
  - The tick that reads the finishing Done: clears the old hold, then the branch sets a new one. Correct.
  - A later non-Done turn end: the hold ends and the normal held turn end path runs. Correct.
  - A turn start: the hold ends and `gateAtBoundary` stays set until the next arrival. Correct.
- `Wait` ungated Done branch:
  - A non-Done outcome finalizes as before. Correct.
  - A Done that is held: no finalize. Correct.
- `Wait` gated Done branch: `holdForReply` runs only when the run is not covered, after the per-turn reset. Correct.
- `Wait` pause-lift branch: `boundaryHeld` is cleared, then the run probes again. Correct.
- `Wait` hold branch, ordered before the pending re-evaluation: a held gated run skips the pending and re-prompt re-evaluation, which is the stated intent. The deadline and liveness checks still run after the chain. Correct.
- `Wait` with a deadline during a hold: `classifyDeadlineExpiry` finds the outputs present and finalizes done. Correct: the hold is bounded.
- `pollEventsTick` `turnStartSeen`: reset each tick and set from the turn starts parsed. Correct.

**Findings**

F1. BLOCKING, design. `internal/shuttleengine/replyhold.go:476-510` with `wait.go:332-334`: the release can finalize a run, or run its gate, while the agent is mid-turn acting on the reply.
- The hold ends early only on `eventsRead || turnStartSeen`.
- `turnStartSeen` comes from the engine's turn-start signals, and for Claude a turn start is only `UserPromptSubmit` (`internal/shuttleengine/claudeengine/signals.go:170-172`).
- The reply reaches the agent through the idle waiter, an `asyncRewake` Stop hook (`claudeengine/settings.go:212`). That hook claims the reply and marks it read before it exits 2.
- The repo's research (`docs/research/creel/creel-hook-channels-2026-10-28/report.md:24`) says this wake "started a turn by itself" and shows only a "Stop hook feedback" line. A Stop-hook wake is not a user prompt, so no `UserPromptSubmit`, and so no turn start, is evidenced for it.
- Once the reply is read, the probe answers false: the question is Replied and no reply is unread. Two quiet probes then release the hold 5–10 s into the reply turn.
  - An ungated run finalizes done mid-turn.
  - A gated run runs `boundaryIdle`, whose `unmatchedTurnStart` finds no turn start, and the gate is evaluated mid-turn.
- The summary's design note, that the second probe "sees that turn start first", depends on a signal this engine path is not shown to write.
- Not verified: whether Claude fires `UserPromptSubmit` on an `asyncRewake` wake. If it does, this finding falls.
- Fix: keep the hold until the reply turn's own turn end is read, by clearing it only on a turn end (`eventsRead`), not on two quiet probes after a reply was read. Or verify and pin, with a corpus test, that the rewake writes a turn start.

F2. MEDIUM, design. `internal/creelengine/verbs.go:1095`: the `asked` predicate is keyed on the strand name alone, so a question from an earlier session that had the same name holds a later run.
- A spawned role's name is its module's constant, so a later round's fixer or reviewer in the same worktree reuses the name.
- `releaseGoneHolder` (`internal/creelengine/sweep.go:215-258`) settles only what the removed strand held, never the questions it sent.
- So an earlier session's question that the orch has not read, or read and left unanswered (chased every 15 min, `chase.go:17`), counts for the new strand.
- Every finishing turn end of the new run is then held until that question settles, or until the run deadline when it stays unread.
- Fix: count only questions posted after the current session began, for example `Posted` at or after the run's start passed with the request, or settle a removed asker's open questions.

Tool calls:
1. `fold.go` status constants and `setStatus`: needed whether a reply to a strand can fold as something other than unread before it is read.
2. `claudeengine/testdata/corpus` grep: looked for a recorded rewake turn, to check for a turn start. None found.
3. `orchcli.NotifyAsker`: needed whether the existing notifier signs as a program the same way, to see whether a program claim from a pane would break the probe. It does sign the same way, so no new risk.
4. `fold.go:150-215` plus a grep for Delivered and Undelivered: needed where pool messages take those statuses, for the reply predicate.
5. Grep of docs and `claudeengine` for rewake turn-start evidence: needed for F1.
