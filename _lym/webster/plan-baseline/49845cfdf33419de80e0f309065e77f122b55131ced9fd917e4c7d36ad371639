# Card 2 — batten-tells-machine

**Edit:**
- `internal/battencli#battenCLI`
- `internal/battencli#battenCLI.specFor`

**Create:**
- `internal/battencli#TestBuildShed_TellsTheMachine`

**Uses:**
- `internal/battencli#newPreDriveReceiver`
- `internal/shedengine#Shed`

**Intent:**
Decision 7: batten's wiring tells its shed `machinename.Name()`, so a batten run's Land Done entry names the machine that landed it.

`battenCLI` gains `machineName func() (string, error)`, documented like its sibling seams: nil selects `machinename.Name`, which only a hand-populated receiver leaves nil.
In `specFor`, the `BuildShed` closure reads the name once per built shed, after `battenrecipe.New` returns, and sets `shed.Machine`.
A read that fails makes `BuildShed` return the refusal `battencli: this machine's name is unknown, so the run cannot record which machine it ran on: <error>; way forward: give the machine a hostname, then re-run lym batten run <slug>`, and no shed is built, so the run never stamps a missing name.
The refusal reaches the operator through `shedverbs`' `BuildShed` error path, the one the cancel-marker refusal of `TestPreDriveCheck` already reaches.
The input that reaches it after every upstream check passes is a host whose `os.Hostname` fails or is empty, which `machinename.Name` returns as an error, since `machinename.FromHostname` refuses an empty name; the test drives it through the seam.

`TestBuildShed_TellsTheMachine`, over `newPreDriveReceiver` with a seeded batten run, has two rows: a `machineName` answering `"neve"` gives a shed whose `Machine` is `"neve"`; one answering an error gives a `BuildShed` error that contains the error's text and `give the machine a hostname`, and a nil shed.

**ImpactSummary:** One nil-defaulting seam on the receiver and one read in the batten shed constructor; loom and darn build no batten shed and are untouched.

**Commit:** 2: Tell batten's shed this machine's name and refuse a run with none
