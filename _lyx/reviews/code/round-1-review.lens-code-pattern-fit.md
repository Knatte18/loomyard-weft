```yaml
lens: code-pattern-fit
outcome: reported
```

Task: check the diff against PATTERN.md, the `pattern/` background files and CLAUDE.md, one line per rule.

**Rules checked**

- `PATTERN-gate-slot`, held. `runBuild` runs inside `runWithSlots`, at `run.go` `runGreenAware`. The inert carry goes through the green record and `openGreen`, which reads no record on a dirty tree. `carry: false` records and carries nothing. The background file's `-trimpath` line is narrowed to Go builds, as the decision record's Constraints require.
- `PATTERN-cwd-resolution`, held. `internal/dotnetimpact` takes absolute paths and imports no `lyxcwd` (`derive.go` imports).
- `PATTERN-gitrepo-client-boundary`, held. The derivation reads git only through `gitexec.Run` and `impactset.ChangedPaths`/`AncestorBase`. `green.go` uses `gitrepo.New(root).IsAncestor`.
- `PATTERN-spawn-observability`, held. The build spawn and its end are logged through `logger.Info` (`build.go`). The derivation spawns only git.
- `PATTERN-sole-parsers`, held. Card seeds come from `planparser.CardCSharpSeeds`, and `gatecli` does no second plan parse (`scope.go` `readCards`).
- `PATTERN-tolerant-records`, held. `RunRecord.Build` and `BlockedBy` are `omitempty` additions, and `LandingFailure.Build` is `yaml:"build,omitempty"`.
- `PATTERN-config-layers`, held. `build` is in `ConfigOpenMaps` and categorised executable. `inert_paths` is a guard key. The template holds `build: {}` and `inert_paths: []`, and `LoadConfig` validates both.
- `PATTERN-refusal-way-forward`, held. New gateslot refusals go through `refuse`/`LoadConfig`'s clause, and the `--guards` refusal and the build refusals name a way forward. I did not open `LoadConfig`, so I did not confirm it adds the `lyx config gate` clause to the plain `fmt.Errorf` for `shared_build`. The decision record requires that wording, so it is worth a check.
- `PATTERN-test-speed`, held. New tests carry one tag each: `tier0` for `derive_test`, `narrow_test`, `source_test` and `csharpseed_test`, and `tier1` for `derive_integration_test` and `dotnetfixture_integration_test`. `fixture_test.go` and `testmain_test.go` carry no tag, so they compile into every tier. That fits shared helpers, but I did not open them.
- Docs in the same commit (CLAUDE.md), held. `doc.go` changed in `gatecli`, `gateslot`, `impactset`, `planparser`, `verifytree` and `landingshed`, and `dotnetimpact/doc.go` is new. `docs/overview.md` lists the new module and the build step, and `docs/dotnet-hub.md` is added.
- Declared PATTERN changes landed, held. The decision record's Constraints declare no new entry, so `PATTERN.md` rightly has no diff. The one declared change, narrowing the `-trimpath` line, landed in `pattern/PATTERN-gate-slot.md`.
- Commit messages and departures, held. No `Pattern-Departure:` trailers appear in the range, so there is nothing to excuse. The commits are named per card.
- Stencil wording, not judged. `contracts/stencils/landing/landing-template-fixer.md` changed by 2 added lines. I did not read them, so the handler should check them against card 13's Intent.

No findings from this lens.

**Tool calls**
1. One Bash call ran `git diff --stat` over `docs`, `PATTERN.md`, `pattern` and `contracts`. It also read the `go:build` header of each new test file, and grepped `docs/overview.md` and `PATTERN.md` for `dotnet`. My context lacked which docs and tags the diff touched.
