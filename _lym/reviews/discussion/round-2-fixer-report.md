# Round 2 fixer-report — discussion

Every change is in `_lym/discussion/decision-record.md`.

## Changes

- F1 (BLOCKING), fixed differently.
  D2 now spawns a fixer only at an evaluation reached at the writer's idle turn boundary.
  At a final evaluation, whether the deadline, liveness or restart finalize, the wrap only waits on a live fixer record and then runs the inner closure once.
  Not made: the proposed "flag the loop sets before finalize's `evaluateGate`". That flag is a `shuttleengine` change, and D2 states that no `shuttleengine` change is made.
  Made instead: the wrap sets the entry's `Final` closure and `MayHold`. `MayHold` lifts only the guard on `Final` on a must-pass entry and the guard on pending (`internal/shuttleengine/gate.go:74-78`, `wait.go:1625`), and finalize runs `Final` in place of `Gate`/`Forked` (`wait.go:1654`).
  `Final` runs the inner closure the run would pick: the inner `Forked` closure when the row's spec runs forked, and the inner `Gate` otherwise.
  The deadline overrun bound is corrected to one fixer timeout plus three inner gate runs, along with the fixer-count bound and the D2 acceptance criterion.
  The Expired-probe restart path is kept; it now says it spawns none.
  The Rationale now says that a fixer is spawned only while the writer is idle.
- F2 (MEDIUM), fixed as proposed, with a different identity.
  The proposed key was the step record's step number, but step records are keyed by trace id (`internal/shedverbs/steprecord.go`), so a re-step gets a new id.
  D8 instead keys a row visit by the row's name, the count of the status `History` entries and the newest entry's `Repeats`, read from `shedrun.StatusFile` when the call starts.
  A finished call appends or folds an entry, a goto appends one, and an interrupted step appends none, so a crash re-step keeps the set and a goto or fresh entry starts an empty one.
  An acceptance criterion was added.
- F3 (MEDIUM), fixed as proposed (`EndRole` option).
  D1 no longer claims "cancel, goto or reset". It names the paths that end the pair's processes through runproc, and registers under the worktree's pair slug.
  D8 adds that a goto ends the fixer roles through `runproc.EndRole` before it moves the run.
  The shed has no reset verb (only `internal/webstercli/reset.go` exists), so "reset" was dropped from D8.
  An acceptance criterion was added: a goto past a row with a live fixer leaves no fixer running.
- F4 (MEDIUM), fixed as proposed.
  In D1, `internal/gatefixer` declares the fixer role set, `gate-fixer` and `verify-fixer`, the latter moved out of `internal/websterengine/verifyfixer.go`.
  D4's Bound tests the caller's role against that set through `agentname.MatchesRole`, so `gatecli` imports nothing of `websterengine`. The Rationale and the D4 acceptance criterion were reworded to match.
- F5 (LOW), fixed as proposed (refuse option).
  D3 refuses `fixer` at config load on every gate key but `card`, naming the key and the way forward. Scope Out's darn-writer line and the D3 acceptance criterion now match.
- F6 (LOW), fixed as proposed.
  D4's card fixer registers under the detached runner's owner, the caller's pair slug. A call with no slug hands off nothing.
  D1 states the same no-slug rule for row callers.
- F7 (LOW), fixed as proposed (no-parent option).
  D4's card fixer renders the parent directive's no-parent variant. A failure it would ask about is left and named in its reply.
- F8 (LOW), fixed as proposed: D7's staleness sentence now covers both the plan and the discussion fixer.

## Self-check

- fix-consistency, MEDIUM (D7 does not test the new no-slug rule). Merged with fix-way-forward LOW and fix-proposal MEDIUM #2. Fixed as proposed: D7's fixer-on conditions and the D7 and D2 acceptance criteria now require a pair slug.
- fix-consistency, LOW (D1's stale "whose run cannot be resolved"): fixed as proposed. It now reads shuttle run, runproc registration or record.
- fix-consistency, LOW (D2's opening "final `Gate` and `Forked`" sits beside shuttle's `Final`): fixed as proposed.
- fix-way-forward, MEDIUM (goto ends only `gate-fixer`, so `verify-fixer` survives): fixed as proposed. Goto now ends each role of `gatefixer`'s fixer role set.
- fix-way-forward, LOW (no-slug rule against D7): merged into the fix-consistency MEDIUM above.
- fix-proposal, MEDIUM (a resume of a failed stop with no history entry keeps the visit key): fixed as proposed. D8 and its criterion now separate a halt that appended an entry from a failed stop that appended none.
- fix-proposal, MEDIUM (F6 against D7): merged into the fix-consistency MEDIUM above.
- fix-claims, MEDIUM (the memoised-verdict sentence holds only for a boundary evaluation that finalizes): fixed as proposed and reworded.
- fix-claims, MEDIUM (a held re-prompt send re-evaluates without spending an attempt, so the fixers are not bounded by `attempts` plus one): fixed with a new rule. The wrap keeps one record per fixer in the visit's set and spawns none past `attempts` plus one. D2's Bound, D8 and a new acceptance criterion say so.
- fix-claims, LOW (the D1 list of runproc callers is incomplete): fixed as proposed. The text now names the group instead of the callers.
- fix-claims, LOW (`originSlug` is unexported in `gatecli`): fixed as proposed. D1 and D4 now name the source, `fabricengine.ReadOrigin`.

## Gate repair

- The gate flagged `citation-path-missing` on D4's role-guard sentence, which cited `internal/gatefixer` without its ` (new)` mark.
  The sentence now reads "the set of fixer roles `internal/gatefixer` (new) owns".

## Disputed

None.

## Deferred

None.
