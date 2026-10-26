```yaml
lens: fix-proposal
outcome: reported
```

fix-proposal lens: I challenged each proposal the fixer carried out as proposed (F1, F2, F3, F4, F6, F7).

No findings.

- **F1:** `beam start` exists at card 10 (`internal/beamcli/cli.go:144`, an operator verb), so `shortcutProblems` resolves the repointed `prime` argv.
  Trailing words go to the last argv, `reed attach`, so the verb's `cobra.NoArgs` gets none.
  Card 26's `TestRemovedVerbs_NamedNowhere` scans `.go` files, stencils and skills, not `shortcuts.yaml`, and card 12 removes `prime` before that card anyway.
- **F2:** I found no case where tagging the pawlengine test files `tier0` is wrong.
  The card's tests run over fakes and an injected sleep, and `TestMain` stays in an untagged support file, which `tiertags_test.go` allows.
- **F3:** `NilSeams` skips `nilLegal` paths, so listing `Pawl.Watch` there means neither wire table may list it.
  "Each remaining `Pawl.&lt;Seam&gt;`" says exactly that.
  `Env.Pawl` is a struct field (`pawlengine.Deps`), so `walkNil` recurses into it.
  Card 17 fills `env.Pawl` only for a pawl run, and both `TestWire_EverySeamFilled` tests wire a loom or batten run, so their reverse check ("lists X but wire() now fills it") does not fire.
- **F4:** `Landing.LandBlockers` matches the dotted path `walkNil` builds, as the other `Landing.*` entries show.
  `ParentWorktree` is a string, which `NilSeams` never reports, so only the reflective `unfilled` map needs it.
  Card 17 fills both only for pawl, so loom's `TestLandingDeps_EveryFieldPopulated` stays right.
- **F6:** `startPlainRun`, `killTreeProcs` and `tailString` are called nowhere in `internal/orchcli` test files outside `TestSmokeOrch_Adopt`.
  `waitForDiag` is still used by `TestSmokeOrch_CompactCycle`.
- **F7:** card 6 is already committed, and `lyx loom validate-plan` passes with `TestNewPawl` and `TestPawlRouting` as Edit targets.

Tool calls:
1. Read `_lyx/plan/17-pawl-wiring.md`, the `battencli` `TestWire_EverySeamFilled` body and a grep for any other `battencli` reflective Deps test. My context lacked card 17's wiring scope, which decides whether the F3 and F4 tables stay right.
2. Read `envkit`'s `nilLegal`, `NilSeams`/`walkNil`, and the run `loomcli`'s `wiring_seams_test.go` wires. My context lacked the dotted-path form and the kinds `walkNil` reports, which F3 and F4 depend on.
