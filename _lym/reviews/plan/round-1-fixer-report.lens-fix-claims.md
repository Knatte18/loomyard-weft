```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: I checked the claims the fix added or rewrote against the code at HEAD.

1. MEDIUM. Card 20's new Bound says "a push after a hand-over stops at this machine's hand-over mark". This holds only when origin holds another machine's takeover entry for the run, which the card 17 route has no reason to find.
   - Evidence: in `internal/fabricengine/takeover.go:117-121`, `takeoverStop` returns nil when `!found || here.Holds(entry.Machine)`. It reads the hand-over mark only after it finds a takeover entry naming another machine.
   - Card 17's route runs on a registry owner reading with no takeover entry on origin, so after such a hand-over the mark stops nothing.
   - The overview's rationale leans on this Bound ("safe, as card 20's Bound says"), so it inherits the gap.
2. LOW. Card 9's new sentence says a clashing current name "is a registration refusal (card 4), so the catch-up stops before any run is read". That is true of the catch-up, but `ThisMachine` also feeds callers no catch-up gates.
   - Evidence: card 9 lists the two closures in `internal/fabriccli#addWeftVerbs` among `ThisMachine`'s callers, and those are hand-run fabric verbs.
   - So the sentence does not bound the leftover `Current` clash in every comparison, though it reads as if it did.
3. Claims I confirmed as true:
   - `HandOver` returns the archive error before `RemovePair` (`internal/pairteardown/handover.go:108-118`).
   - `HandOverTips` "pushes no branch but the owed code tip, and no tag but the owner archive tags"; its parent reset is local (`internal/fabricengine/handover.go:55-65`).
   - `machinename.FilePath` reports ok false when no variable is set, so the empty `XDG_CONFIG_HOME` and `HOME` fixture reaches the refusal (`internal/machinename/machinename.go:42-51`, `internal/standalonestate/standalonestate.go:173-185`).
   - Card 19's re-sign keeps the existing parameter order of `commitRunIntent` (`internal/tendercli/runintents.go:123`).
   - `startDobbyStep` already warns on a refused entry.
   - `refuseByAct` runs first in `startFirstPass`.
   - `parentlock.Store.HeldBy` and `parentlock.Dir` exist as card 21 names them.

Tool calls:
- Read `FilePath`, `machineFile` and the user config dir resolution: my context lacked how an empty `HOME` resolves.
- Read the `commitRunIntent` signature and `HandOverTips`' doc: my context lacked the current parameter order and what the hand-over pushes or resets.
