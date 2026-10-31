# A long gate run answers within the cache lifetime, so the waiting agent keeps its prompt cache warm

`gate-run-keeps-cache-warm` · Note · enhancement, gate, shuttle, config

## Seen (operator, 2026-10-31)

Gates on OSL-1046 take more than five minutes, a Webster gate there about 11, and an agent blocked on one comes back to a cold prompt cache: the whole context is written to the cache again.
`lym gate test` already caps its wait for a free slot and answers "run the same command again" (exit 75), so that wait keeps the cache warm; the run itself, once the slots are held, blocks the call to its end.
`lym shuttle await` is the capped keep-warm wait for a file, but an agent may not start a gate in the background, so nothing writes a file it could await.
The cap is a constant, `shuttleengine.DefaultAwaitCap` (250 s), shared by `lym shuttle await` and the gate's slot wait, and sized for the 5-minute prompt-cache lifetime; a session may run with a 1-hour cache lifetime instead.

## Idea

- The gate run gets the same cap as the slot wait: when the cap passes with the tiers still running, `lym gate test` answers that the run goes on and to run the same command again, while the tiers carry on in a process of their own.
  The same command again joins that run, waits up to the cap once more, and prints the result when the run has ended.
  Agents learn nothing new: the gate rule already says to re-run the same command in a turn that does nothing else.
- The cap is config, not a constant, read by every keep-warm wait: the slot wait, the run wait and `lym shuttle await`.
  It is set by where in a run the wait happens, not in one place: a darn run may want another cap than Webster, so it can differ per recipe, row or role, with a hub-wide default beside the engine's prompt-cache lifetime (operator, 2026-10-31).
- Keeping the cache warm pays only up to a break-even wait: each re-run costs a cache read of the whole context, about a tenth of its price, while a cold return costs one cache write, about 1.25 times its price at the 5-minute lifetime, so past roughly ten re-runs, some 45 minutes, a cold cache is cheaper.
  A wait known to be longer than that, such as a gate that takes two hours, blocks to its end without re-runs (operator, 2026-10-31).
  The expected length can come from the gate's own records of earlier runs of the same gate key on the machine, which `lym gate report` already totals; the break-even itself is config too.
- Find the other waits that block an agent past the cache lifetime and give them the same capped form; the driver blocked on `lym shed step` through a long producer is the first to check.
