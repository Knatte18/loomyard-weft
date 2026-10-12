# Card 13 — `lyx shuttle open` opens a named LLM session with no turn in its own window

**Create:**
- `internal/shuttleengine#OpenResult`
- `internal/shuttleengine#Runner.Open`
- `internal/shuttleengine#openMarkerPath`
- `internal/shuttleengine#openerFakeEngine`
- `internal/shuttleengine#openerFakeEngine.OpenLaunch`
- `internal/shuttleengine#TestRunner_Open`
- `internal/shuttlecli#shuttleCLI.openCmd`

**Edit:**
- `internal/shuttlecli/cli.go`
- `internal/shuttleengine/doc.go`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`
- `internal/shuttlecli#TestRunCLI_Run_FlagValidation`

**Uses:**
- `internal/shuttleengine#OpenSpec`
- `internal/shuttleengine#SessionOpener`
- `internal/shell#posixShell.Exit`
- `internal/shuttleengine#Runner.SetPause`
- `internal/shuttleengine#fakeReed`
- `internal/shuttleengine#fakeEngine`
- `internal/shuttleengine#newFixture`
- `internal/shuttlecli#shuttleCLI.sendCmd`

**Intent:**
D12: the orch, or the operator, opens an extra named Claude session (one for exploring the repo, say) in its own window through shuttle, with its session id on the strand so the watchdog's name repair covers it.

`Runner.Open(spec)` in a new file `internal/shuttleengine/open.go` (package `shuttleengine`, with a file comment), in order:

1. The told-path error, as every runner verb returns it.
2. An empty `spec.Role` is refused (`shuttle: open: the role must not be empty`); `lyx shuttle open` refuses an empty `--role` itself, so only a direct caller reaches it.
3. With a pause geometry told (`SetPause`), `pauserecord.ReadCoverage` of its record for its slug; a covered worktree is refused with `pauserecord.CoveredError(entry, slug, "re-run lyx shuttle open")`, which names the pause and the way forward, before anything is created.
   An unreadable record is a Warn and the open goes on, as `lyx shuttle send` does.
4. An engine without `SessionOpener` is refused (`shuttle: open: the engine cannot open a session with no prompt`).
5. `OpenLaunch(spec, cfg)`; its error is returned.
6. The exit marker `openMarkerPath(anchorPath, token)`, `<anchor>/.lyx/shuttle-open/<token>.exited` (the `.lyx` segment through `lyxdirs`), for a fresh random token, with its directory created.
   It lives beside, never inside, the run-directory root, so no run sweep or step stop reads it.
7. Through `shell.ForGOOS()` (`PATTERN-shell-mechanics-seam`), the strand's command chains, after the provider line, the marker write, reed's detached self-removal and the shell's exit:
   `Chain(launch.Cmd, Touch(marker), Invoke("lyx") + " reed remove --name " + EnvRef(LYX_STRAND_NAME) + " --detach", Exit())`, and the resume command is the same chain over `launch.ResumeCmd`.
   `lyx` in the pane resolves to the binary that spawned it through reed's pane prelude (`PATTERN-pane-binary-resolution`).
   When the operator ends the session the marker is written and the detached remover removes the strand, so its window closes; when the self-removal fails the shell's exit leaves a dead pane bound until `lyx reed remove` or a pair's teardown, and `Resume` drops the strand through its done-when marker instead of relaunching it (reed's side is pinned in card 6).
   A pair teardown ends the reed session first, which ends the pane's shell before the chain goes on, so teardown never runs the self-removal.
8. `reed.AddStrand(AddSpec{Role: spec.Role, Segment: string(spec.Segment), Cmd, ResumeCmd, SessionID: launch.SessionID, Display: {Anchor: own-window}, DoneWhen: [marker]})`.
   The role comes from the caller and is numbered by reed when held (`explore`, then `explore-2`), never `NameOverride` and never a literal (`PATTERN-agent-name`).
   The session id on the strand is what the watchdog's name repair reads; `Resume` relaunches the strand through its resume command, under the same name and session id, in its own window.
9. It logs `shuttle: opened session` at Info with the strand, role, session id and marker (`PATTERN-spawn-observability`), and returns `OpenResult{StrandGUID, StrandName, SessionID}` without waiting.

It writes no run directory, no `run.json` and no output file, so `lyx shuttle state` and the hold notices never cover an opened session.
Bound: one interactive session per call, in the caller's own worktree under its settings, never a turn; nothing caps the count, since a pair's teardown ends every opened session and in the prime one lives until its provider exits or the operator removes it.

`internal/shuttlecli/cli.go`:

- `shuttleCLI` gains `configDirs configengine.Dirs`, set by `PersistentPreRunE` to the same `fabricengine.ConfigDirs(layout)` it already loads both configs from.
- `Command` registers `c.openCmd()`, and the file comment lists `open.go` among the verbs.

`shuttleCLI.openCmd()`, in a new file `internal/shuttlecli/open.go` (package `shuttlecli`, with a file comment):

- `Use: "open"`, `Short: "open a named llm session with no prompt in its own window"`, audience `operator` (the orch acting for the operator), and a `Long` saying `--role` is required, the session takes no turn, writes no run record, closes its window when the session exits, and is refused under a pause.
- Flags: `--role`, `--model` (a model spec, empty for the provider default), `--segment` (the loom segment its window is colored by).
  `--role` is not marked required with cobra, whose refusal is a plain error rather than a JSON envelope; `RunE` refuses an empty `--role` first with the JSON error `--role is required`, as `run` refuses a missing `--output-file`.
- A non-empty `--model` is parsed with `modelspec.Parse` before anything else reads config, then resolved against `modelspec.LoadRegistry(c.configDirs)`; a parse, load or resolve error is a JSON error naming `--model`.
  The resolved model, `effort` and `version` fill `OpenSpec`.
- `c.runner.Open(spec)`; an error is a JSON error, and success prints `{"guid", "name", "sessionId"}`.
- `clihelp.ShouldAbort` comes first, as in every `RunE` (`PATTERN-cli-cobra`).
- The verb takes the shuttle group's existing wiring, the `PersistentPreRunE` that resolves the layout, both configs, the reed engine, the runner and the pause geometry once for every shuttle verb.
  The decision record's constraint names `internal/cliwire` (`PATTERN-cliwire-sole-wiring`), but no shuttle verb is wired through it today; giving one verb a second wiring path would split the group, and moving the whole group onto `cliwire` is outside this task's scope, so this card records the deviation here rather than taking it silently.
- The operator command index (`lyx help index`, loaded by the `ly:cli` skill) lists it through its `operator` audience.

`internal/shuttleengine/doc.go` gains an "Opened sessions" paragraph on `Runner.Open`, its chain, the marker, the pause refusal and the absence of a run record.
`docs/overview.md`'s shuttle bullet appears twice on one physical line, the second copy fused onto the first after "documentation."; the card drops the older first copy (the one without the 60-second dialog notice and the `sed`/`rm` guardrails), so one bullet remains, and that bullet lists `lyx shuttle run|open|interrupt|send|state` and adds one sentence: `open` starts a named session with no turn in its own window, which closes when the session exits.
The bullet that gives `reed`, `shuttle` and `loom` each a user-facing CLI changes `lyx shuttle run|interrupt|send|state|await` to `lyx shuttle run|open|interrupt|send|state|await`.

Tests:

- `openerFakeEngine` in `fakes_test.go` embeds `fakeEngine` and returns a scripted `Launch` (or error) from `OpenLaunch`, recording the spec.
- New `TestRunner_Open` in `internal/shuttleengine/open_test.go`, one table:
  - `adds an own-window strand by role`: `AddStrand` is called once with `Role: "explore"`, no `NameOverride`, `Segment: "review"`, `SessionID` the launch's, `Display.Anchor` `own-window` and one done-when path under `<anchor>/.lyx/shuttle-open/` ending `.exited`; `Cmd` equals the dialect's `Chain` of the launch line, the marker touch, the detached `lyx reed remove --name` over the strand-name reference, and `exit`; `ResumeCmd` is the same chain over the resume line; the result carries the fake strand's guid and name and the session id; and the run-directory root holds no run directory.
  - `an empty role is refused before anything is created`.
  - `a live pause covering the worktree is refused`: a pause record with a live hub-wide entry, told through `SetPause`: the error contains `way forward` and `re-run lyx shuttle open`, and no `AddStrand` call.
  - `an engine without the capability is refused`: a plain `fakeEngine`, no `AddStrand` call.
  - `a failed open launch is returned`: no `AddStrand` call.
  - `a second open of the same role passes the role again`: two `Open` calls with `Role: "explore"` each call `AddStrand` with `Role: "explore"` and an empty `NameOverride`, so reed numbers the second `explore-2` by its existing rule (pinned by `internal/reedengine#TestStrandNameLocked_Names`), and their markers differ.
- The self-removal's `lyx reed remove --name <full name> --detach` resolves a full `<shortname>:<slug>:<role>` name, which `internal/reedcli#TestStrandNamingAndList` already pins by removing a strand by its full name; reed's window end on that removal is pinned by card 4's `RemoveEndsTheOwnWindow`.
- `TestRunCLI_Run_FlagValidation` gains a `build func(c *shuttleCLI) *cobra.Command` row field (nil meaning `c.runCmd`) and rows `OpenMissingRole` (`c.openCmd`, no args, `wantErr: "--role is required"`) and `OpenInvalidModelSpec` (`c.openCmd`, `--role explore --model "opus[nope"`, `wantErr: "--model"`), each refused with one envelope before the nil runner is touched; its doc comment names the open rows.
- `cmd/lyx/testdata/test-budget.yaml`: raise `internal/shuttleengine` by one.

**ImpactSummary:** Adds one runner verb and its CLI; `cli.go` gains one field and one registration, and no existing run path changes.

**Commit:** 13: lyx shuttle open opens a named session with no turn in its own window
