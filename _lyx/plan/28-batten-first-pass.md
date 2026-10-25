# Card 28 — batten-first-pass

**Edit:**
- `internal/battencli#battenCLI`
- `internal/battencli#Command`
- `internal/battencli#battenVerbTexts`
- `internal/battencli#battenCLI.driveRun`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#TestBattenRun_RefusesWhileAPauseCoversTheSlug`
- `internal/battencli#TestArmAt_StartGuardRefusesBeforeAnyWrite`
- `internal/adoptcli#hubDepsAs`
- `internal/adoptcli#adoptDeps`
- `internal/beamcli#hubLandDeps`
- `internal/beamcli#landDeps`
- `internal/beamcli#landCmd`
- `internal/battenshed/doc.go`
- `internal/battenshed/template.yaml`
- `docs/overview.md`

**Delete:**
- `internal/battencli#battenCLI.runInWindow`
- `internal/battencli#battenCLI.openWindowInPrimeSession`
- `internal/battencli#windowSite`
- `internal/battencli#ResumeInWindow`
- `internal/battencli#windowRefusal`
- `internal/battencli#windowNamePrefix`
- `internal/battencli#openTerminalFlagName`
- `internal/battencli#TestRunInWindow`
- `internal/battencli#TestWindowFlagIsOnTheRunVerbOnly`
- `internal/battencli#TestWindowSite`

**Create:**
- `plan:internal/battencli#StartFirstPass` -> `func StartFirstPass(location *lyxcwd.Location, slug string) (map[string]any, error)`
- `plan:internal/battencli#ensureParentDobby` -> `func ensureParentDobby(prime *lyxcwd.Location, slug string, live func(dobbyengine.Geometry) (bool, error), spawn func(dobbyengine.Geometry) error) error`
- `plan:internal/battencli#TestStartFirstPass` -> `func TestStartFirstPass(t *testing.T)`

**Uses:**
- `plan:internal/hubgeom#DobbyGeometry`
- `plan:internal/dobbyengine#Live`
- `plan:internal/dobbyengine#SpawnDetached`

**Intent:**
Retire `--window` and the hidden `--open-terminal`, since dobby starts every run, and start a first pass detached where a verb used to open a window (D4).
The run verb loses both flags, the window opener and the launcher seams; the open_window setting alone decides the Open-Window row, and `wire`'s setting lookup loses the forced-on answer.
`StartFirstPass` arms the run as `ResumeInWindow` did, runs the same pre-checks (the covering pause, the status check and the owner pre-flight) and returns their refusal as an envelope whose `ok` is false, then starts `<lyx> batten run <slug>` detached in its own process group with the hub root as its working directory, logs the spawn at Info, and calls `ensureParentDobby`; its envelope names the started pass's pid and the parent worktree whose dobby drives it.
`ensureParentDobby` resolves the run's parent worktree through `hubgeom.ParentOf` (the prime for none), and starts that worktree's dobby through `spawn` when `live` reads it not live; production passes `dobbyengine.Live` and `dobbyengine.SpawnDetached`, and `battenCLI` gains the `passSpawn`, `dobbyLive` and `dobbySpawn` seams a hand-populated receiver fakes, nil selecting the production ones; a failure is returned and the caller warns, naming `lyx beam start` as the way forward.
`lyx adopt` and `lyx beam land`, `ResumeInWindow`'s callers, take `StartFirstPass` through their `ResumeRun` and `StartLanding` seams; `beam land`'s help and `landDeps`' doc say the landing starts as a detached first pass driven by the parent's dobby.
`driveRun`, after the pass's envelope, calls `ensureParentDobby` and logs a failure at Warn, so a hand-run `lyx batten run <slug>` runs one pass in the foreground and then makes sure the parent's dobby is live.
The run also removes the board note `batten-window-default`, whose idea is moot, by running `lyx board remove '{"slug":"batten-window-default"}'` once while doing this card; it writes no other board entry.
The run verb's help, `cli.go`'s package comment, `battenshed/doc.go`'s start paragraph, the template's `open_window` comment and `docs/overview.md`'s batten sentences drop `--window` and say a run is one pass, started by its parent's dobby or by hand.
Tests: `TestStartFirstPass` (tier0, parallel) over a hand-populated receiver with fake `passSpawn`, `dobbyLive` and `dobbySpawn`: a done slug and a slug a pause covers return an envelope with `ok` false naming their way forward and spawn nothing; a startable slug spawns one pass and, with the parent's dobby read not live, spawns that dobby once, and with it read live spawns none; a failed dobby spawn still returns the started pass's envelope with a warning naming `lyx beam start`.
`TestBattenRun_RefusesWhileAPauseCoversTheSlug` loses its window opener and launcher and its `runInWindow` arm, keeping card 26's `driveRun` row; `TestArmAt_StartGuardRefusesBeforeAnyWrite` runs `run <slug>` without `--window` in its envelope row and asserts the same refusal with no seed written; the window tests go with the window code.

**ImpactSummary:** The run verb loses two flags and its window body; adopt's and beam land's one seam each take the detached first pass, and two tests that drove the window path lose it.
