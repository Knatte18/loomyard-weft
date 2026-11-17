```yaml
lens: fix-affected-set
outcome: reported
```

Task: for each finding whose premise is a set (F3, F5, F8), rerun its search and check that the fix covers every member.

No findings.

Left unchecked: none. F1, F2, F4, F6, F7, F9, F10 and F11 have no set as their premise.

- F3: four places in `internal/shuttleengine/codexengine/prepare_test.go` (lines 117, 229, 270 and 335) build whole `Prepare` launch lines through `buildLaunchCmd`. Those are `TestPrepare_Launch`, `TestPrepare_SandboxMode`, `TestPrepare_ExpandsHomeInWritableRoots` and `TestPrepare_InstallsGuardsFromItsInputs`, which card 10 already lists. Card 6's `TestPrepare_SubagentTerms` builds whole lines too, and card 10 now lists it as well.
  - The two `codexCommand` calls in `command_test.go:58` and `open_test.go:133` build opened lines, which card 10 leaves with no cap, so they are unaffected.
  - Card 9's `TestPrepare_UserSetup` and card 10's `TestPrepare_InstructionFiles` assert parts of the line, not the whole line, so they are unaffected.
  - The set is covered.
- F5: the parent's own search found four `RenderRequest` literals: `seam.go:80` (`requestOf`), `heddlecli/verbs.go:553` (`renderInto`), `render_test.go:45` (`f.request()`) and `render_integration_test.go:48`. Card 4 now sets `Root` in all four, and its Edit list names both test builders. The set is covered.
- F8: in the code, only the doc comment on `Target` (`internal/heddleengine/target.go:46-47`) describes `RendersInto` as naming file targets alone. The other `RendersInto` sites are code. The one prose site, the `Show` refusal at `render.go:341`, is already rewritten by card 2. The set is covered.

Further tool calls:
- grep of codexengine's tests for whole-line builders (`launchLine{`, `buildLaunchCmd(`, `codexCommand(`): my context held the plan cards but not the test files, which F3's set needed.
- grep of non-test files for `RendersInto`: my context held only `target.go:40-50`, and F8's set needed every prose site.
