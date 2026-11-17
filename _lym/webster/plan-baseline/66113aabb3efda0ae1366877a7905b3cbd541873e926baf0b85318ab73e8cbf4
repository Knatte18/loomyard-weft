# Card 6 — `lym beam status` shows the Bolt hold

**Edit:**
- `internal/beamcli#adoptSection`
- `internal/beamcli#adoptSectionOf` -> `func adoptSectionOf(hold boardengine.Hold, own bool) *adoptSection`
- `internal/beamcli#dobbyStatus`
- `internal/beamcli#statusResult`
- `internal/beamcli#statusCmd`
- `internal/beamcli/doc.go`
- `internal/beamcli#TestAdoptSectionOf`
- `internal/beamcli_test#TestStatus_PrintsThePrimesDobbyAndEachLocalBeams`

**Uses:**
- `internal/boardengine#Hold`
- `internal/boardengine#ReadOwners`
- `internal/boardengine#Registry.ReadsAsHere`
- `internal/boardengine#WriteHold`

**Intent:**
The `adopt` section of `lym beam status` reads the Bolt hold, not the adopt record.
`dobbyStatus` reads the hub's local Bolt through `ReadOwners` and, when a hold stands, sets `Adopt` from the first hold, `own` being whether its machine `ReadsAsHere` for this machine's identity.
A board that does not read joins the failures as `adopt hold: <err>`, as the record read did.
`adoptSection` carries `Kind`, `Machine`, `At`, `Subjects`, `FixTask` and per-subject `Tries` (both only under `adopt-fix`), and `Rerun`, the `lym adopt <subjects>` command, only for this machine's own hold.
Its lines read `adopt hold (<kind>) by <machine> since <time> over <subjects>`, then under `adopt-fix` `fix task <slug>, tries <subject> <n>...`, then for its own hold `way forward: <rerun>`.
The hold's subjects render root as `root`.
`statusResult`'s comment and `statusCmd`'s long help name the hold section; doc.go drops the adopt record and `--discard`.

`TestAdoptSectionOf` (tier 0) rows: `AnAdoptHoldOfAnotherMachineShowsKindMachineTimeAndSubjectsAndNoRerun`, `AnAdoptFixHoldShowsItsFixTaskAndTries`, `ThisMachinesHoldNamesTheRerun`.
`TestStatus_PrintsThePrimesDobbyAndEachLocalBeams` (tier 1) plants a hold with `WriteHold` in place of the stopped adopt record and asserts the section's kind, machine and subjects; its undecodable-record step asserts an undecodable `board.json` joins the error instead.

**ImpactSummary:** beamcli's status verb only; the envelope's `adopt` object changes shape, and no other package reads it.

**Commit:** 6: Show the Bolt hold in `lym beam status` in place of the adopt record
