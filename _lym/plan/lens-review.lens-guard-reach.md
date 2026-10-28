```yaml
lens: guard-reach
outcome: reported
```

Task: guard-reach review of the plan's guards, refusals, branches and test rows.

**Findings**

1. **BLOCKING — Card 11, D4/D5 bell after a ring.** The 5-second grace path makes the bell due 5 seconds after every ring, which bypasses the 30-second ring-answer window.
   - `internal/creelengine/pool.go:346` `answerLocked` deletes the answered waiters from `s.waiters` at once.
   - `daemon.awaitOutcome` then defers `Waiter.Cancel`, and Card 11 records the waiter's end time in `Cancel`.
   - So right after a ring the strand has no live waiter. Its "last waiter ending" time is the ring itself, and `bellGrace` is satisfied 5 seconds later.
   - Result: the bell is typed while the woken turn may still be running, before its Stop claims and before the window ends.
   - D4 says the bell comes only "when neither a claim nor a busy reading follows a ring within 30 seconds".
   - Card 11's own row "a ring answered by a claim within the window is not followed by the bell" passes only if the claim lands within 5 seconds.
   - The grace path needs to exclude a set that has already been rung.

2. **MEDIUM — Cards 10 and 11, the ring hold.** Card 11 says "A newer set clears the mark, so its bell may be typed once more."
   - Card 10 keeps the ring's time on the same `wakeMark` and holds `NoTurn` false "until a turn end recorded after" the ring.
   - Mail that arrives in a woken turn's blocked chain is a newer set. If that set clears the mark's ring time, the hold is gone and the waiter started by the last blocked Stop rings mid-chain. That is the case D4's hold exists to stop.
   - The ring time has to survive a set change. No row in Card 10 or Card 11 covers a new set arriving mid-chain.

3. **MEDIUM — Card 13, D2/D3 exit on a Stop.** A subagent call "exits 0" on every event, Stop included.
   - Card 14's merged Stop command reads exit 0 as "a block was written" and appends no turn-end record.
   - D2 says `hook-read` exits 0 on a Stop "exactly when it wrote a block".
   - If the subagent guard is ever reached on a Stop (the Constraints call D3 a guard on Stop, and `agent_id` is also set for in-process teammates), the turn end is silently dropped.
   - On a Stop, the subagent branch should exit non-zero so the turn end is recorded. No row runs a Stop input that carries `agent_id`.

4. **LOW — Card 7, D4 "a draft does not hold the ring".** `Runner.SessionIdle` has an early return when operator activity holds a ready reading: `internal/shuttleengine/session.go:249-251` returns `IdleProbe{Reason: hold}`.
   - Card 7 fills `NoTurn` and `TurnEndedAt` only "for a ready or held reading" built from `readiness`, so this branch returns `NoTurn` false.
   - An idle hook strand whose operator was recently active is then never rung, although no turn runs.
   - The card neither sets `NoTurn` on this branch nor states it as an intended hold.

5. **LOW — Cards 8 and 9, the pane-mail branch.** `PaneMail` is true only for a strand with a non-empty provider that has no renderer.
   - `internal/shuttleengines/shuttleengines.go:49-50` registers only `claude`, and shuttle launches only registered engines.
   - So the pane path for posts cannot be reached in production today. Only the unregistered test providers (`codex`, `script`) reach it.
   - This is consistent with the added decision, but the intents don't say so; Card 8's "Bound" sentence covers only hand-added Codex strands.

Tool calls:
- grep of `internal/shuttleengines/shuttleengines.go` registry and `internal/creelengine/pool.go` `answerLocked`/`Waiter.Cancel`/`addWaiterLocked`: my context lacked which engines are registered and whether a ring removes the waiter from the live registry.
