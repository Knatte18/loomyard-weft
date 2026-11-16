```yaml
lens: interactions
outcome: reported
```

Task: interactions lens, finding where the record's decisions act on the same state and checking each pair, criterion and constraint.

**Pairs checked**
- D1 × D2, rejected push reads the takeover before any replay: held.
- D1 × D2, Finalize order (parent branch push, then transition, then board `done`): held. Both state the same order.
- D1 × D5, owed-push retry under a hold's `hold` slug entries: **broken** (F1).
- D1 × D5, admission before commit against a hold written after it: held.
- D1 × D7, `lym tender resume` pays deferred detached pushes only: held.
- D1 × D9, the stop's bound rests on synchronous pushes, and an owed push is picked up by the catch-up at the next dobby start: held.
- D2 × D3, the takeover entry's machine-name comparison goes through the registry: held.
- D2 × D4, in-loop hand-over against the catch-up's step 4 for a run owned elsewhere: **broken** (F2, F3).
- D2 × D7, a pair left by an in-loop hand-over against the teardown condition: **broken** (F2).
- D2 × D9, Finalize cut by the stop: held. D2 states the same-machine resume.
- D3 × D10, owner filter through the registry: held.
- D4 × D5, catch-up step 5 slug entries from pause intents against `hold` entries ("a slug with a live entry gets no `hold` entry"): held.
- D4 × D10, the catch-up removes a hub's launcher when nothing there is owned here: held.
- D5 × D6, green clear drops the `hold` entries; `adopt-fix` exempts the fix task; the landing past merge acts under a hold: held.
- D5 × D6, the upgrade's first-read hold against the re-run of `lym adopt` of its subjects: held.
- D5 × D7, the teardown condition excludes held pairs: held.
- D5 × D9, the stop's hub entry lifted by identity (`tenderengine.Lift` matches scope, caller and time) leaves the `hold` entries alone: held.
- D5 × D10, a start under a hold prints the hold and the act rule waits: held.
- D6 × D2, the adopter's rejected takeover push re-reads hold and owner: held.
- D6 × D4, steps 2 and 3 reset the moved runs before step 4's catch-up pushes local commits: held.
- D8 × D3 and D5, shared `boardengine`, mapping at the printing boundary only: held.
- Criteria D1 to D10 and Constraints against their decisions: held, apart from the three findings below.

**Findings**

F1. MEDIUM, design. D5 × D1, `_lym/discussion/decision-record.md:155-157` against `:50-51`.
D5 writes a `hold` slug entry for each run "the act rule answers `wait` for under the hold".
A run whose push is owed answers `wait` (`internal/runfsm/act.go:85-86`, `PushOwedOffline` is checked first), so it gets a `hold` entry.
D1's owed-push retry "waits at the gate like any network call, so a pause holds it" (`decision-record.md:51`), so on a non-adopting machine an owed transition cannot reach origin while the hold stands.
The adopter then adopts from origin's older transition, and the owed transition is archived rather than pushed.
That cuts against D1's "whole state on GitHub" purpose, and the record says neither that this is intended nor which wins.
Fix: either exempt an owed run's push from the `hold` entry, or state in D5's bound that the hold freezes owed pushes and that D2's archive covers them.

F2. MEDIUM, design. D2 × D7 (× D4), `decision-record.md:72-79`, `:227-228`, `:131-133`.
D2's in-loop hand-over on a rejected push ends processes, archives and pushes tags, but leaves the local pair standing.
D7 tears down only pairs "with … no other machine owning it", and the pair is no orphan because it has a board entry and a batten record.
Only D4's catch-up removes such a pair, and that runs only after a sleep, a dobby start or a hand pass.
On a dobby that stays up, a handed-over pair stands indefinitely with nothing reporting it, which is the exact case D7's rationale names (`:246`).
Fix: have D2's hand-over remove the pair through `pairteardown`'s local mode as D4 step 4 does, or have D7 list it.

F3. LOW, consistency. D4 step 4 against D2's hand-over, `decision-record.md:131-133` against `:78` and `:100`.
D4 step 4 "archives the local tips as D2 does", but it omits D2's archive-tag push and D2's one notice naming any dropped pause or cancel intent.
An intent owed offline under D1 (`:54`) on a machine that sleeps through a takeover is then dropped by the catch-up with no notice.
That breaks D2's bound that the notice names it to repeat on the owner machine.
Fix: make D4 step 4 push the tags and post D2's notice.

No tool calls made.
