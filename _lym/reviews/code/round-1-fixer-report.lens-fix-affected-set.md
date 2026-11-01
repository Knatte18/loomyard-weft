```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches for F1 and F7 and check the fix covers every site. F2 to F6 don't rest on a set, so I left them unchecked.

**F1 — MEDIUM.** Changed text: `crucible/orchestrator-prompt.md:119-124`. The fix covers only that one file. The same `-tags tier2` and `-tags tier3` smoke commands remain in two sibling crucible files. Under the run-time selector they run tier0 only, so every smoke test skips and the run ends green, which is the defect F1 names.
- `crucible/README.md:132-134` has `go test -tags tier2 ./internal/<module>cli/... -run Smoke -v -count=1`, and lines 132-133 say to use `-tags tier3` for an LLM-driving module.
- `crucible/README.md:144` has `go test -c -tags tier2 -o "$SCRATCH/smoke.test.exe" ...`, the compiled-binary form that needs `LYM_TEST_TIER` exported at run time.
- `crucible/review-prompt-template.md:135, 149, 150` tell reviewers to run `go test -tags tier2 <MODULE CLI PACKAGE> -run Smoke ...` and `go test -tags tier3 ... -run <ExactTestName>`, and to treat `-tags tier2` / `-tags tier3` as the live-substrate invocation.
- Other `-tags tier` hits are out of scope. The `shellguard` and `planparser` tests are data rows. `internal/gatecli/doc.go` is in the review's Rejected list. The `cmd/testtiming` text uses `-tags` as that command's own flag. `docs/benchmarks/running-tests.md` drives `testtiming`.

**F7 — LOW.** Changed text: the stale-wording edits in `internal/loomcli`, `websterengine`, `hubforge`, `hubgeom`, `battencli`, `tmuxkit` and the pattern files. The fixer reworded exactly the sites the review listed, but the listed sites are a sample of a larger set. The same build-tag wording remains:
- Testmain headers saying "It carries no build tag, so it compiles into every tag set" (same kind of file as the five fixed):
  - `internal/statuspane/testmain_test.go:2`
  - `internal/standalonestate/testmain_test.go:2`
  - `internal/commentlint/testmain_test.go:2`
  - `internal/loomshed/testmain_test.go:2`
  - `internal/verifyrun/testmain_test.go:2`
  - `internal/dotnetimpact/fixture_test.go:2`
  - `internal/verifytree/testmain_test.go:2`
  - `internal/gitrepo/internal/gitoracle/testmain_test.go:4`
  - `internal/fabricengine/refusalcheck_helper_test.go:2`
  - `internal/configcli/configcli_helper_test.go:2`
- Test-file headers saying "untagged" or "integration-tagged" for a tier:
  - `internal/websterengine/template_test.go:3`, `contractevidence_test.go:3`, `runwrites_test.go:4`
  - `internal/fabricengine/rawworktree_test.go:4-5`, `fabric_test.go:6-8`, `slug_test.go:2`, `revert_test.go:1`, `recordswritelock_test.go:1`, `mergestate_test.go:7`
  - `internal/burlercli/wiring_test.go:350, 876`
  - `internal/treadleengine/engine_test.go:7`
  - `internal/fabriccli/argsarity_test.go:3`

I left the `untagged` word in `contracts/specs/loom-plan-spec.md:631-632` and `internal/loomshed/doc.go:33` out. That word is a plan-gate token the code still matches, not stale wording.

Calls made:
1. Grep for `-tags tier` across md, sh, yaml and go files, outside `_lym`, `.lym` and `.scratch`: the F1 set, which my context didn't hold.
2. Grep for `untagged`, `tagged test`, `either tag`, `every tag set` and related build-tag wording, outside those same directories and `docs/benchmarks`: the F7 set, which my context didn't hold.
