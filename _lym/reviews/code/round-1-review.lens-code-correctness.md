```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness.** I traced each branch the diff adds or changes in the creel engine, orchroute and the reply paths, running one concrete input per branch.

**Branch list (case run → verdict)**
- `chaser.run`:
  - Start probe, then the first tick charges one interval: correct.
  - A stopped clock backs off: 1m, 2m, 4m, then 8m (`chaseProbeCeiling`): correct.
  - A wake during a running wait keeps the tick's deadline: correct.
  - `retire()` against `wakeChaserLocked`, both under `scheduler.mu`: no lost chaser.
  - The store wake hook is called after `s.mu` is released (`pool.go:164-170`), so `scheduler.mu` then `store.mu` has no lock inversion: correct.
- `ageChases`: a hold by another sender skips the message, and a hold by the message's own sender lets the clock run: correct.
- `stepChases`: the message was read at t=0 with no restart, so it is due at 15m.
  - Chase 1 goes at 15m, chase 2 at 30m, and it is settled `no-reply` at 45m.
  - A step error after `delete(chaseBase)` retries at the next tick: correct.
- `seedChaseClock` and `repairChases`:
  - A restart with the step overdue sets `chaseBase` to the consumed clock, so the next step falls one interval after the restart: correct.
  - A crash between a chase's post and its `chased` record is repaired from `Message.Chases`, with no third chase: correct.
- `postChaseLocked`: the body carries no sender text, and the CREEL name does not become a known program mailbox (`knowProgramMailbox`): correct.
- `settleUnansweredLocked`:
  - `holder-gone` and `no-reply` both journal, notice and answer the wait.
  - A late reply still wins in `settledOutcomeLocked`: correct.
  - The sender-notice path calls `delivery.Strands()` under the store lock; see F1.
- `noticeUnansweredLocked`:
  - A listed strand sender gets a notice.
  - A program that has never read, a strand no longer listed, and EXTERNAL get none: correct.
- `releaseGoneHolder`:
  - A holder with only a read, marked message and no other mail or inject is found through `sweepSubjects`, because `isChaseable` is included.
  - The message is settled `holder-gone` and no chase is sent: correct.
- `Reply`:
  - A reply to a chase from its holder answers the chased message, with the wait keyed on the chased id: correct.
  - The former-holder guard refuses and names the way forward: correct.
  - A reply to a CREEL notice is refused: correct.
  - The holder check now runs before validation: correct.
- `Forward`: a chase is refused before the "already read" case and names `--reply`: correct.
- `PostWith`/`post`/`storePost`: `Wait` implies the mark, the server dispatch has the same behaviour as before, and `OutcomeUnanswered` carries the message: correct.
- Fold:
  - `applyPosted` sets `ChaseMark` and `Chases` from the chase's posted record, and `Replied` from a reply to a marked message: correct.
  - `applyCarried` and `carryRecords` round-trip `ReadAt`, `ConsumedSeconds`, `MarkSeconds` and `Chase`, and carry program readers: correct.
  - `prune` drops `chased` entries together with their messages: correct.
- `MessageLine`/`LineFits`: validation uses the longest, marked form, so any message that fits is also fine unmarked: correct.
- `orchroute.StrandOf` and `Asker` (read up to the pair/beam split; the end of `Asker` was cut off in my context and not read): no defect seen.

**Findings**

F1 — LOW, design — `internal/creelengine/chase.go:235` (called via `settleUnansweredLocked` at `chase.go:224`, from `stepChases` and `releaseGoneHolder`)
`noticeUnansweredLocked` calls `s.delivery.Strands()` while holding the store mutex.
Every other `Strands()` call in the store (`readStrands` in `verbs.go:170`, `:582` and `:751`) runs outside `s.do`, and `sweep.go:114` and the chaser probe likewise run it lock-free.
A slow tmux or reed listing therefore blocks every store verb.
Fix: read the strands before entering `s.do` in `stepChases` and `releaseGoneHolder` and pass them in, or make the notice conditional on a snapshot taken outside the lock.

**Bookkeeping, not a finding**
`chaseWritten` and `chaseBase` entries are never deleted once a message settles. This is a small leak and I rate it a NIT, so I did not file it.

**Tool calls**
1. `grep` of `wakeLocked`, `SetWakeHook`, `do` and every `delivery.Strands()` call in `internal/creelengine`: my context lacked how the wake hook is invoked relative to `s.mu` (to rule out a lock inversion) and where else `Strands()` runs under the lock.
