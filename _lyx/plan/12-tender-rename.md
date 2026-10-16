# Card 12 — tender-rename

**Rename:**
- `internal/pausecli/cli.go` -> `internal/tendercli/cli.go`
- `internal/pausecli/cli_test.go` -> `internal/tendercli/cli_test.go`
- `internal/pausecli/deps.go` -> `internal/tendercli/deps.go`
- `internal/pausecli/deps_test.go` -> `internal/tendercli/deps_test.go`
- `internal/pausecli/doc.go` -> `internal/tendercli/doc.go`
- `internal/pausecli/gate.go` -> `internal/tendercli/gate.go`
- `internal/pausecli/testmain_test.go` -> `internal/tendercli/testmain_test.go`
- `internal/pauseengine/doc.go` -> `internal/tenderengine/doc.go`
- `internal/pauseengine/pause.go` -> `internal/tenderengine/pause.go`
- `internal/pauseengine/pause_test.go` -> `internal/tenderengine/pause_test.go`
- `internal/pauseengine/report.go` -> `internal/tenderengine/report.go`
- `internal/pauseengine/resume.go` -> `internal/tenderengine/resume.go`
- `internal/pauseengine/resume_test.go` -> `internal/tenderengine/resume_test.go`
- `internal/pauseengine/testmain_test.go` -> `internal/tenderengine/testmain_test.go`

**Edit:**
- `cmd/lyx#newRoot`
- `cmd/lyx/testdata/test-budget.yaml`
- `docs/overview.md`
- `pattern/PATTERN-cli-cobra.md`

**Create:**
- `plan:cmd/lyx#runCLISeams` -> `var runCLISeams []func(io.Writer, []string) int`
- `plan:cmd/lyx#runCLIInSeams` -> `var runCLIInSeams []func(string, io.Writer, []string) int`

**Intent:**
`internal/pausecli` becomes `internal/tendercli` and `internal/pauseengine` becomes `internal/tenderengine`, the module's own name; `internal/pauserecord` keeps its name and role, since it is a leaf imported widely whose name still describes what it holds (D1).
This is the last card, so every earlier card targets glyphs that resolve against the tree as it stands.
It is a pure rename, apart from naming the seam test's two slices: no behavior changes, and the existing tests carry it.

Each pair moves with `git mv` first, per the plan's rename mechanic, followed by only the surgical edits:
- package clauses `package tendercli` and `package tenderengine`, and the package doc comments' opening `Package tendercli` and `Package tenderengine`;
- the import of `internal/pauseengine` in `tendercli` and every `pauseengine.` qualifier there;
- `tendercli`'s log messages, which start `pausecli:`, now start `tendercli:`.
  Error texts keep their `pause:` prefix, which names the verb, and the shortcut spellings `lyx pause` and `lyx resume`.
`InstallNetworkGate` moves with the CLI package, and `newRoot`'s pre-run calls `tendercli.InstallNetworkGate`; its module list mounts `tendercli.Command()`.
`cmd/lyx/seamsignature_test.go` lists `tendercli.RunCLI` and `tendercli.RunCLIIn` in place of the `pausecli` ones.
Its two blank `var _` slices become the named vars `runCLISeams` and `runCLIInSeams`, with their elements unchanged, so the file has members a card can name; the compile stays the assertion.
Each slice's comment says "The var below" in place of "The blank identifier below", and is otherwise unchanged.
`test-budget.yaml` replaces the rows `internal/pausecli: 4` and `internal/pauseengine: 5` with `internal/tendercli: 4` and `internal/tenderengine: 5`, sorted in place.

`docs/overview.md`'s package tree lines and the tender module bullet name `internal/tendercli` and `internal/tenderengine`.
`PATTERN-cli-cobra.md`'s deviation line says `tendercli` uses `tenderengine` and mounts one root command, `tender`.
`tendercli/doc.go` keeps its documented reason for resolving the hub through `lyxcwd` itself, the fixed dependency set that bars `cliwire` (`PATTERN-cliwire-sole-wiring`).
`docs/research/codex-feasibility.md` is a dated measurement report and keeps its `pausecli/deps.go` mention.

A grep for `pausecli` and `pauseengine` outside `_lyx/` and `docs/research/` finds nothing after this card.

**ImpactSummary:** Two packages move to their module's name with no behavior change; their only importers outside themselves are `cmd/lyx`'s `newRoot` and seam-signature test, both edited here.

**Commit:** 12: tender: rename pausecli and pauseengine to tendercli and tenderengine
