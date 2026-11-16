# Round 2 fixer-report: discussion decision record

Every finding's premise held against the code (`internal/fabricengine/coalesce.go`, `internal/pauserecord/record.go`, `internal/machinecli/set.go`, `internal/machinename/machinename.go`, `internal/runfsm/act.go`).
All changes are in `_lym/discussion/decision-record.md`.

## Changes

- F1, fixed differently in part: D1 gains a bullet saying that every detached pair push keeps `PushPairAnchored`'s order.
  The order moves into `coalescePushSides`: records first, the code side only once records land, and a takeover read on rejection with no code side.
  A D1 criterion now says an anchor change's push in a taken-over pair leaves origin's code branch unchanged.
  The proposal called `coalescePushSides` the step "both the synchronous and the detached pushes run". That is not made: `PushPairAnchored` runs its own `pushSide` step (`internal/fabricengine/pushanchored.go:91-140`), not `coalescePushSides`.
- F2: D5 gains a bullet: a `bolt-hold` entry names the hold's machine, and `pauserecord.CoveredError` words it as a hold with D5's way forward, never `lym tender resume <slug>`. A D5 criterion is added.
  The `Coverage.Held` clause was made, then removed after the self-check. Its display claim is false; see Self-check.
- F3: D3 comparisons resolve a name to a registry entry and read it as this machine only on an id match. An unregistered machine reads no registered name as its own.
  A machine now registers at its first catch-up or dobby start after the upgrade.
  A copied `machine.json` is refused, its way forward `lym machine name <new>`, which runs the same check and writes a fresh id. The copied-file case is added to the bound, with a criterion.
- F4: every writer of `machine.json` keeps the id. A machine with no config directory is refused at registration, naming the variable. The lost-file bound now says its own hold reads as another machine's and is moved with `--take-hold`.
- F5, fixed differently: `--forget-former` lists the owners, takeover entries and unfinished cancel intents under the dropped name, with `lym adopt` as the way to take each back.
  The bound now states the real reach: from the other machine's registration on, every record under the clashing name reads as that machine whether this machine forgets the name or not.
  The proposed refusal was made, then replaced after the self-check, because it could never clear. The other machine's legitimate owners carry that name, and this machine, being unregistered, cannot `lym adopt`: D6 step 1 refuses on a refused registration. See Self-check P2/WF1.
- F6: D2 says the pair push's caller hands `fabricengine` this machine's current and former names, and the takeover read compares against that set. A D2 criterion is added.
- F7: D5's ordering claim is dropped. The hold no longer holds an owed push, and a `bolt-hold` entry is dropped once the run's push turns owed.
  D4 step 4 now pushes owed commits records first when origin has no takeover entry. D2's bound now covers an owed push that lands after the takeover.
- F8, fixed differently after the self-check: D6 step 2 decides whether a run moved from the hold's recorded owner before the adopt, a new hold field kept across a re-run and by `--take-hold`, together with whether a takeover entry names another machine.
  The proposed key, "owner before step 1", was made and replaced because it skips every run on a killed adopt's re-run. A D6 criterion is added.
- F9: the Open risks bullet is removed and the section reads "None."; "(Open risks)" is cut from D9's rationale.
- F10: D9's bound and rationale say "pushed or owed", and the bound adds that D4's catch-up pushes an owed transition.
- F11: D2's mark lives under the hub's own `.lym`, keyed by slug. D4 step 4 writes it once its archive lands, and D2's own hand-over now writes it at the same point, before the local removal.
- F12: D5's upgrade bullet states the order: intents and hold pushed, then the legacy entries dropped.
- F13: the D2 criterion is reworded as proposed.
- F14: Scope's runfsm exception is reworded as proposed.
- F15: D6's Kept paragraph names `lym adopt check`, run through `fronttask`'s `adopt-check` row, and what it reads and does on green.
- F16: D7's orphan bullet names the first-sight file under the hub's `.lym`, in `internal/lymdirs`.
- F17: D6 step 1 and `--take-hold` define the same subjects as the same set, and refuse any other set, an overlap included.

## Self-check

- consistency 1, LOW: deferred. The parent's Added-after-Discussion entry still says "pushed state". It is the parent's decision text, which the parent changes with `lym loom decision replace`.
- consistency 2, LOW: fixed. The D5 owed-push criterion now allows the hand-over archive.
- consistency 3, LOW: fixed. D2 writes its mark once the archive tags land, before the local removal.
- consistency 4, LOW: fixed. The registry setter writes at a registration or an owner write, and D4 step 1 registers.
- consistency 5, LOW: fixed by the F5 rework. `--forget-former` no longer refuses, so "after that verb, the registration lands" holds.
- claims 1, MEDIUM: fixed. The `Coverage.Held` display claim is removed, since its only reader is the dobby's owner probe, which retires (`internal/battencli/dobbyreads.go:179`).
- proposal 1 (BLOCKING, F8 breaks a killed adopt's re-run): fixed differently, as F8 above.
- proposal 2 (BLOCKING) and way-forward 1 (MEDIUM), merged (the `--forget-former` refusal never clears): fixed differently, as F5 above.
- way-forward 2, MEDIUM (an adopt writes no takeover entry for an owned run): fixed by the F5 rework, which drops the refusal.
- proposal 3, MEDIUM (forgetting a name trips the copy check): fixed. A forgotten name moves to the file's forgotten names, which the copy check reads as this machine's own.
- way-forward 3, LOW (rename keeps the id versus writes a fresh one): fixed. `lym machine name` runs the copy check itself, writing a fresh id only on a hit and keeping it otherwise.
- way-forward 4, LOW, and proposal 5, MEDIUM, merged (a registration refused at a dobby start): fixed. Such a refusal stops the catch-up or dobby start with a notice and a `lym beam status` line, and the D3 bound states that a machine with no config directory runs no dobby until the variable is set.
- proposal 4, MEDIUM (a non-owner pushes in D4 step 4): fixed. The exception, a catch-up's push of owed commits made while this machine owned the run, is recorded in Notes for the plan writer as an addition to `PATTERN-owner-machine`'s background.
  It was first added to the PATTERN change's rule text. The gate refused it there twice: once as a continuation line (`pattern-change-malformed`), and once for taking the rule text over the 280-character cap (`pattern-change-over-cap`). It was therefore moved to the background note.
- proposal 6, LOW (a stale orphan first-sight time): fixed. The time is also dropped once the pair stops being an orphan.
- proposal 7, LOW (a re-run first read writes the hold again): fixed as a bound in D5's upgrade bullet: the re-run writes the hold again, and a re-run of `lym adopt` clears it.

## Disputed

None.

## Deferred

- Self-check consistency 1: the parent's Added-after-Discussion entry on the hub stop still says "every run is left in a pushed state", while D9 now says pushed or owed. That entry is the parent's recorded decision, and the parent changes it with `lym loom decision replace`.

## Rulings

None.
