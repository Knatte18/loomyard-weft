# Card 14 — role-file-command-index

**Create:**
- `plan:internal/orchcli#newIndexSource` -> `func newIndexSource(strands strandOps, run func(bin string) (string, error)) func() (string, error)`
- `plan:internal/orchcli#runHelpIndex` -> `func runHelpIndex(bin string) (string, error)`

**Edit:**
- `contracts/stencils/orch/orch-template-role.md`
- `internal/orchengine#RenderRoleFile` -> `func RenderRoleFile(stencilsDir, path, index string) error`
- `internal/orchengine#renderFile` -> `func renderFile(stencilsDir, name, path string, values map[string]string) error`
- `internal/orchengine#RenderNoteTemplateFile`
- `internal/orchengine#ResumeContext` -> `func ResumeContext(p Paths, stencilsDir, index string, now time.Time) (string, error)`
- `internal/orchengine#Watcher`
- `internal/orchengine#NewWatcher` -> `func NewWatcher(session Session, cfg Config, paths Paths, stencilsDir string, skills []string, index func() (string, error), clock Clock) *Watcher`
- `internal/orchengine#Watcher.startClearing`
- `internal/orchengine#Watcher.startAutoReload`
- `internal/orchengine#Watcher.reloadAfterCompaction`
- `internal/orchengine/prompt_test.go`
- `internal/orchengine/resumecontext_test.go`
- `internal/orchengine/watcher_test.go`
- `internal/orchengine/watcher_compact_test.go`
- `internal/orchengine/watchloop_test.go`
- `internal/orchcli#orchCLI`
- `internal/orchcli#Command`
- `internal/orchcli#orchCLI.startCmd`
- `internal/orchcli#orchCLI.watchCmd`
- `internal/orchcli#orchCLI.resumeContextCmd`
- `internal/orchcli/start_test.go`
- `internal/orchcli/watch_test.go`
- `internal/orchcli/cli_test.go`
- `contracts/stencils/markers_test.go`
- `contracts/stencils/claims_test.go`

**Prosa:**
- `internal/orchengine/doc.go`
- `internal/orchcli/doc.go`

**Uses:**
- `internal/clihelp#RenderIndex`
- `internal/clihelp#AudienceOperator`
- `internal/stencil#TopLevelMarkers`
- `internal/reedengine#StrandStatus`
- `internal/orchcli#orchStrands`
- `internal/orchcli#strandOps`

**Intent:** Put the operator command index into the orch's role file on every launch and reload (D4).
The role stencil gains a `## Commands` section holding the marker `{{.command_index}}` and one sentence saying the list is complete and `lyx <command> --help` gives the details; its header comment stops saying the stencil carries no markers and still asks no agent to load a skill.
`RenderRoleFile` takes the index string and fills the marker; before filling, it reads the stencil's top-level markers through `stencil.TopLevelMarkers` and refuses with an error naming `orch-template-role` when `command_index` is absent, so an operator-edited copy without the marker fails the render by name instead of silently rendering no index; `renderFile` takes the values map, and `RenderNoteTemplateFile` passes nil.
`ResumeContext` takes the index and passes it to every role render it makes.
`Watcher` gains an index source, `func() (string, error)`, told through `NewWatcher`; the three render sites call it before `RenderRoleFile`, and each returns to idle with a reason carrying the error when the source or `RenderRoleFile` fails, never a failed tick (Shared Decision `added-watcher-decisions-yield-to-d4`).
`startClearing` and `reloadAfterCompaction` already go idle on a render failure; `startAutoReload`, which today returns that error as a failed tick counted against the loop's consecutive-error cap, goes idle with the reason like them, which is the only change to the watcher's reload steps.
`orchcli.startCmd` and `resumeContextCmd` compute the index in-process as `clihelp.RenderIndex(cmd.Root(), clihelp.AudienceOperator)`, since each runs as a fresh process of the binary the pane names.
`watchCmd` cannot, because the watcher outlives a deploy and may run from another binary than the pane's; it builds its source with `newIndexSource(c.strands, c.indexRunner)`, where `orchCLI` gains the `indexRunner` field that `Command`'s pre-run sets to `runHelpIndex` and tests set to a fake.
`newIndexSource` returns a closure that, at each call, reads the strands, finds the orch strand through `orchStrands`, and errors when there is none, when its `LyxBin` is empty (recorded before the field existed) or when `os.Stat` of that path fails, each message naming the way forward `lyx orch stop`, then `lyx orch start`; otherwise it returns `run(path)`.
`runHelpIndex` runs `<bin> help index` with `exec.Command`, logs the spawn and its completion through `logger.Info` (`PATTERN-spawn-observability`), and returns the output or the error with the command's stderr; it never looks `lyx` up on `PATH` and never runs the watcher's own executable.
Tests, all tier 1 with no spawn: `prompt_test.go` renders the role file with an index and asserts the `## Commands` section carries it, and adds a row where a role stencil without the marker fails naming `orch-template-role`; `resumecontext_test.go` passes an index and asserts it reaches the file; `watcher_test.go`'s `watchEnv.newWatcher` passes a fake source and the clear-cycle render test asserts the index lands in the role file, while the failing-render test and the auto-compaction reload test gain a failing-source case each, both asserting idle with the reason and no tick error; `watcher_compact_test.go`'s reload-after-compaction test asserts the index too; `watchloop_test.go`'s two `NewWatcher` calls pass a fake source; `start_test.go`'s fresh-launch test asserts the rendered role file carries `## Commands` with the operator index of the test root; `cli_test.go`'s `newTestCLI` installs a fake runner and the resume-context test asserts the index in the file; `watch_test.go`'s `TestWatch` gains rows for a strand with a recorded path (the fake runner's output lands in the role file at a reload render) and for a strand without one (the watcher goes idle with the reason naming `lyx orch stop`).
`markers_test.go` gains the `orch-template-role` row with `command_index`, and `claims_test.go`'s role-stencil entry adds `## Commands` to the themes it must carry.
The orchengine doc notes the role file's `## Commands` section and that the watcher is told its index source, and the orchcli doc notes which binary each render site reads.

**ImpactSummary:** Three orchengine functions re-signed with every caller in orchengine, orchcli and their tests on this card; the role file gains a section and the watcher a told source, with the stencil marker check the only new refusal; `startAutoReload` goes idle on a render failure instead of failing the tick.
