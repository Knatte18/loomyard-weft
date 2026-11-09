```yaml
lens: way-forward
outcome: reported
```

Way-forward lens on the adopt plan: for each refusal, stop or recovery step the cards add, I checked the next step it names and whether that step can work from the state the refusal leaves.

**MEDIUM — D2 / Card 3 `Read` and Card 8 `discard`: the way forward from an undecodable record does not progress.**
- Card 3 makes an `adopt.json` that does not decode an error naming `lym adopt --discard`.
- Card 8's `discard` removes "an unfinished record; a done record, or none, is left". To know whether a record is unfinished it has to read it, and `adoptrecord.Read` fails on this file. No card says `discard` removes a record that does not decode.
- The caller's and the worker's `judgeStanding` read the same record, so every `lym adopt` stops on that error. `lym beam status` (Card 9) joins the error and points at the same `--discard`.
- The named next step leaves the hub stuck. `TestDiscard` and `TestRead` have no row where a discard follows an undecodable record.
- Fix: `discard`, holding `adopt.lock`, removes a record that does not decode, and `TestDiscard` gains that row.

**MEDIUM — D3 / D2: the cure for a held-back stop can leave it held back.**
- Card 6's `stepDone` ends `stopped` when `Inputs.HeldBack` names a project. Its way forward is the re-run with `--keep-local` or `--drop-local`.
- On that re-run, Card 7 restarts at `reconcile` and says it keeps "the done steps and stored inputs". No card says `stepReconcile` or `stepPairs` clears `HeldBack`, `Adoptable` or `OwnerStood` before refilling them; Card 6 only says `stepPairs` "fills" them.
- If the old `HeldBack` survives, `stepDone` stops again even after `--keep-local` cured the stop.
- `TestRunWorker`'s row "a resume adding `--keep-local` restarts at `reconcile`" never asserts that the record then ends `done`, so the recovery path goes untested.

**LOW — D1 / Card 8 `follow`: an exit with no record names no cause.**
- A worker can exit without writing a record: its own `validateRequest` refuses after the board changed, or it loses `adopt.lock`.
- `follow` then returns an error naming the re-run and `lym beam status`. `beam status` shows no adopt section, because there is no record.
- The worker's refusal envelope is in `adopt.out`, and the error does not name it. A refusal that repeats on every re-run stays invisible.
- Fix: name `adopt.out` in that error.

**LOW — D3/D8 / Card 6 `heldBeamWayForward`: a named step may have nothing to act on.**
- The new text names `lym tender resume <slug>` and `lym loom resume` for a landing-held beam whose parent another machine owns.
- `pauseEntries` gives a beam slug an `adopt` entry only by converting an existing own-scope `owner` entry. A beam with no such entry has nothing for `lym tender resume <slug>` to lift.
- That step then lifts nothing or refuses, which `tenderengine.Resume` does is not stated. The card should name the tender resume only when the beam's slug is in `Inputs.Paused`.

**LOW — Card 8: the `--discard` misuse refusal has no reaching test.**
- `--discard` with a subject or another flag is refused naming `lym adopt --discard` alone.
- No planned test reaches it: `TestDiscard` drives `discard()`, not `Command`'s flag check.

**LOW — Card 7 `runWorker`: a branch that cannot occur.**
- `runWorker` turns a `follow` verdict into a refusal naming `lym beam status`.
- A worker holding `adopt.lock` cannot see a record whose worker reads alive with matching start ticks, so no input reaches this branch.
- It should be stated as a wiring guard rather than a prescribed refusal.

Tool calls:
- I grepped `boardengine`'s `Recorded`, which my context lacked, to check whether a same-owner re-run still starts the check. It does: `ownerset.go:62` says Recorded is the owner as read after the push, so no finding.
- I read the `adopt_test.go` row `ReRunWithHoldsStanding…` (lines 651–662) for the same question, because my context had only its name.
