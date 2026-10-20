# Card 5 — derived-command-gate-test

**Edit:**
- `internal/impactset#buildCommand`
- `internal/impactset#Derivation`
- `internal/impactset/doc.go`
- `docs/overview.md`
- `internal/impactset#gateCommand`
- `internal/impactset#TestCommandFor`
- `internal/impactset#TestDerive_OverARepository`
- `internal/loomshed#TestVerifyGate_RoundScenario`

**Uses:**
- `internal/testkit/lyxbin#Build`

**Intent:** Move the derived command's test and guard steps onto `lyx gate test --tags integration` (D3), so inside a verify they inherit the held slot, each publish their prebuilt `lyx` under one content-derived store path and drop the trace id, keeping Go's test cache warm.
This changes the command both darn's writer gates (Card 7) and loom's round gate at Webster-Burler-Fix and Merge-Burler-Fix run; nothing else of loom changes.

`buildCommand` keeps `go build ./...` and the three `go vet` steps, and writes the test step as `lyx gate test --tags integration <set>` and the guard tests outside the set as one step, `lyx gate test --tags integration <outside guard packages> -- -run '^(<names>)$'`, with the packages in sorted directory order and the names sorted and deduplicated, as webster's card-gate guard step already does (`internal/websterengine/guardstep.go`), joined by `&&` as before.
One guard step builds `lyx` once, where a step per package would build it once per package.
The steps use only flags `lyx gate test` already takes, so the `lyx` a task pane finds on its `PATH` runs them.
`Derivation.Command`'s doc and `doc.go`'s command paragraph name the new steps.
`doc.go`'s package sentence, which names only the Webster-Burler-Fix round gate as the command's user, and `docs/overview.md`'s impactset line, which calls it the round gate's command, are restated to name loom's round gate and darn's two writer gates.

`gateCommand` renders the new test step, and `TestCommandFor`'s rows and `TestDerive_OverARepository`'s one-package row expect the new test step and the single guard step's spelling.

`TestVerifyGate_RoundScenario` runs derived commands for real, which now call `lyx`.
It builds `lyx` with `lyxbin.Build` and puts its directory first on `PATH` through `t.Setenv`, so it drops `t.Parallel` and its doc comment names `PATH` as the process-global state it sets.
Its fixture repo lies outside every hub, so each `lyx gate test` there runs unslotted and builds no `lyx`, since the fixture has no `cmd/lyx`.

**ImpactSummary:** one command string changes; Derive's fallbacks, set and length cap are unchanged, the round gate runs the same tests through the gate verb, and one loomshed scenario test becomes serial.
