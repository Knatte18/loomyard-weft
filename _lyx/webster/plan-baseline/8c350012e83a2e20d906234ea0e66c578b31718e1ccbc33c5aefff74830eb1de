# Card 36 — dobby-board-tidy

**Edit:**
- `plan:internal/dobbyengine#Deps`
- `plan:internal/dobbyengine#daemon.tick`
- `internal/dobbyengine#dobbyengineAllowedImports`
- `contracts/stencils#entries`
- `contracts/stencils#roleOpeningStencils`
- `internal/dobbyengine/doc.go`

**Create:**
- `contracts/stencils/beam/beam-template-board-tidy.md`
- `plan:contracts/stencils#BeamTemplateBoardTidy` -> `var BeamTemplateBoardTidy []byte`
- `plan:internal/dobbyengine#BoardTidyRole` -> `const BoardTidyRole = "board-tidy"`
- `plan:internal/dobbyengine#TidyPromptInput` -> `type TidyPromptInput struct`
- `plan:internal/dobbyengine#TidyPrompt` -> `func TidyPrompt(stencilsDir string, in TidyPromptInput) (string, error)`
- `plan:internal/dobbyengine#TidySpec` -> `func TidySpec(prompt, reportPath string, model shuttleengine.ModelChoice, timeout time.Duration) shuttleengine.Spec`
- `plan:internal/dobbyengine#daemon.countLandings` -> `func (d *daemon) countLandings(now time.Time) error`
- `plan:internal/dobbyengine#daemon.tidyStep` -> `func (d *daemon) tidyStep(ctx context.Context, now time.Time) error`
- `plan:internal/dobbyengine#TestCountLandingsAndTidy` -> `func TestCountLandingsAndTidy(t *testing.T)`
- `plan:internal/dobbyengine#TestTidyPrompt` -> `func TestTidyPrompt(t *testing.T)`

**Uses:**
- `internal/dobbyengine#Config`

**Intent:**
Spawn the board-tidy agent every N landings, taking over what the orch did after a landing (D10, D9's dropped `landed`).
`countLandings` reads, on each tick, the batten history of every run the status files name with the dobby's worktree as parent, whatever the run's state, and records each run's Land Done as counted in the state record, so a landing a hand pass or a pass a dead dobby left made is counted and a restart double counts none; a dobby that starts with no state record, or meets a fresh record card 4 wrote over an undecodable one, counts every Land Done already there as seen, so run records left from before it count nothing and a rewrite forces no spurious tidy.
Accepted bound: a fresh record loses a tidy in flight and the landed slugs since the last tidy; that agent still ends on its own at `TidySpec`'s timeout, its report is not posted, and the next tidy may spawn while it runs.
Accepted bound: a landing whose records `lyx batten cleanup --apply` removed before dobby read them is not counted, so a tidy may come later than every `board_tidy_every` landings.
At every `board_tidy_every` landings (0: never) `tidyStep` spawns the tidy agent with the landed slugs since the last tidy, unless one is in flight: a fresh shuttle run through `Deps`' shuttle in a window of the dobby's worktree's reed session, model `sonnet[medium]` resolved by the wiring, role `BoardTidyRole` (the beam module's constant, `PATTERN-agent-name`), its prompt `TidyPrompt` filled from the stencil read at call time through `internal/stencilstore` (`PATTERN-stencil-ownership`), its parent directive naming the nearest live orch so a question reaches an LLM that can answer it, the edit directive, and one output file, the report under the marker directory.
`TidySpec` sets the launch spec's `Segment` to `segmentcolor.Coordinator`, since the tidy agent acts for the beam's driver as the arbiter's spec does, and its `Skills` to `ly:board`, the one skill the agent needs to read and amend the board through `lyx board` (`PATTERN-parent-directive`'s spawn-color and role-skill rules); `dobbyengineAllowedImports` gains `internal/shuttleengine`, `internal/stencilstore`, `internal/segmentcolor`, and `internal/parentdirective`, `internal/editdirective` and `internal/stencil` for the two directives and the stencil fill `TidyPrompt` makes as `loomcli`'s arbiter prompt does, the imports this card brings in.
The stencil's brief is today's orch rule: read each landed commit and its run's decision record against the board's notes, through `lyx board` only; make each obvious amendment itself, such as dropping a sentence the landing made stale or noting that part of a note is now built, with body drafts only in `.scratch/`; and write the report: what it changed, and each note a landing superseded whole or changed in scope or design, with why; it removes no note, changes no scope and files or promotes no task.
The tidy in flight (strand, report path, deadline) is in the state record, so a restart or a re-exec spawns no second; once the report exists dobby posts its path to the nearest live orch (`board tidy after <n> landings: report <path>; put each removal or scope change to the operator`) and ends the agent through the shuttle stop verb, and past `board_tidy_timeout_min` it ends the agent the same way, keeps that tidy's landed slugs in the state record so the next tidy names them with its own, and posts `board tidy after <n> landings gave no report; its slugs carry to the next tidy: <slugs>`.
Bound: one tidy agent at a time per dobby, bar the fresh-record case above.
`contracts/stencils`' registry lists the stencil, and `roleOpeningStencils` maps `board-tidy` to it, so the parent- and edit-directive scans cover it.
Tests (tier0, fakes, temp dirs, parallel): `TestCountLandingsAndTidy` rows: five Land Done entries across runs with `board_tidy_every: 5` spawn one agent naming the five slugs; a restart with the same records spawns none and counts none twice; a first start over old records counts none; `board_tidy_every: 0` spawns none; a report written posts its path once and stops the agent; a deadline passed stops it and posts the no-report line naming its slugs, and the next tidy names those earlier slugs with its own. `TestTidyPrompt` fills the stencil with the slugs, the report path and the parent directive naming the orch, and a row asserts `TidySpec`'s `Segment` is `segmentcolor.Coordinator` and its `Skills` are `ly:board`, as `internal/shedarbiter#TestSpec` does for the arbiter.

**ImpactSummary:** One more engine step with its stencil and registry rows; the stencil scans and the registry test cover the new role.
