```yaml
lens: affected-set
outcome: reported
```

Lens: affected-set. For each change a card makes, I traced what reads or asserts it and which cards cover that set.

**Per-change sets and covering cards**

- **Card 1, `ParseOwnerRepo` shapes.**
  - Set: `TestParseOwnerRepo`, the callers `fabriccli` and `hubissues`, `ownerRepoPath`'s doc.
  - Covered by card 1.
- **Card 2, `beam.pull_request`.**
  - Set: `dobbyengine` `Config`/`ConfigSpec`/`LoadConfig`, `template.yaml`, `TestLoadConfig`, configreg's `wantLocalDenied`, `dobbyengine/doc.go`.
  - Covered by card 2.
- **Card 3, board `pull_request` field.**
  - Set: `Task`, `normalizeTask`, `replayFields`, `UpsertPayloadKeys`, `scopeChanged`, `storeVerbs`' help, `TestHelpSchema_LeafCommands`, `TestRunLock`, boardengine doc.
  - Covered by card 3.
- **Card 4, `pr_reviewer` key and `pr-review` gate key.**
  - Set: landing `Config`/`template.yaml`/tests, configreg, gateslot `template.yaml`/`goReadGates`/`templateDefaults`/doc, envkit `fixedGateLists`.
  - Covered by card 4.
  - Possibly also: other files that list the gate keys (see finding F5).
- **Card 5, empty `OutputFiles` for an unawaited run.**
  - Set:
    - `stopOutcome` and `stopRecorded` (`internal/shuttleengine/stop.go:18`, `:75`)
    - `Spec.validate`
    - `wait.go`'s package-header precondition "no *Run with an empty OutputFiles ever exists" (`wait.go:73-76`) and its restatements at `wait.go:1498` and `:1530`
    - `run.go:609-611` `doneWhen`, which gives reed an empty `DoneWhen`
    - `attach.go:536`/`:597` `allOutputFilesExist(spec.OutputFiles)`
    - `waitshadow.go:155`
    - `sessionread.go:124`, already guarded with `len&gt;0`
  - Card 5 covers only `stopOutcome`, `validate` and its own tests.
- **Card 6, publish shell-guard row.**
  - Set: `shellguard` `Rows`/`Switch`, claudeengine's builders and every listed test, the `cmd/lym/norecoverydeny_test.go` scan over `Forms` (the row has no `Forms`, so it holds), the golden (byte-identical).
  - Covered by card 6.
- **Card 7, new `pullrequest` package and shedrun paths.**
  - Set: `TestPathConstructors`, the module table in `docs/overview.md`.
  - Covered by card 7.
- **Card 10, new stencil.**
  - Set: `entries` (the name-to-default registry, `contracts/stencils/stencils.go:450-456`), `roleOpeningStencils`, `TestRegistry_MatchesOnDiskTree`.
  - Card 10 lists `registry` (the type with `Names`, `stencils.go:597`) instead of `entries`.
- **Cards 11 and 12, retiring the stub, `pr_review_setting` and `loom.pr_review`/`darn.pr_review`.**
  - Set:
    - `entries_prreview.go`, `entries_prreview_test.go`, `conformance_test.go:88`, `registry_test.go:108`
    - `shedbuild/build_engines_test.go:64-65`
    - both recipes' rows and header comments
    - `shedrecipe/doc.go:51-54` and `:94` ("An auto-accept value, like the PRReview flag, is a bool")
    - landingshed `doc.go:1`/`:3`, `prreview_test.go:3`
    - `docs/overview.md:269`/`:578`
    - `loomengine` `config.go:421-422`/`:552` and `template.yaml:43`
    - `darnengine` `config.go:60-61`/`:99`, `template.yaml:12`, `doc.go:3`
    - configreg_test `:77`
  - Covered by cards 11 and 12, except `shedrecipe/doc.go:94`.
- **Card 13, Finalize text.** The PR-Review row closing on `Continue` makes `finalize.go:162`/`:523` and landingshed `doc.go:178` false. Card 15 covers them.
- **Card 14, `InterruptPolicies` comment "spawns nothing".** Set: `interruptpolicy.go:43`. Covered by card 14.
- **Card 17, `childAtBeamAwait` → `childAtWaitingRow`.**
  - Set: `battencli/wire.go:635`/`:650-652`, its test, `battenshed/doc.go`.
  - Covered by card 17; no other reference exists.

**Findings**

- **F1 — MEDIUM, design, card 5 (`05-shuttle-unawaited-session`).**
  - Card 5 breaks the precondition shuttle's completion-signal rule rests on: an unawaited `*Run` with an empty `OutputFiles`.
  - It updates only `stopOutcome`. It leaves these false or unchecked:
    - the precondition stated at `wait.go:73-76`, and restated at `wait.go:1498` and `:1530`
    - `attach.go:536` and `:597`, where `allOutputFilesExist` is vacuously true for an empty list
    - `waitshadow.go:155`
    - `run.go:609-611`, which passes `spec.OutputFiles` as reed's `DoneWhen`; the card states no behaviour for reed's resume of an unawaited strand
  - Evidence: `internal/shuttleengine/wait.go:73-76`, `run.go:609-611`, `attach.go:536`, `stop.go:18-22`.
- **F2 — MEDIUM, consistency, card 10 (`10-pr-review-stencil`).**
  - The card edits `contracts/stencils#registry`, but the list it adds to is `entries` (`stencils.go:450-456`); `registry` is the `stencilstore.Registry` type at `stencils.go:597`, which does not change.
  - Fix: replace the target `contracts/stencils#registry` with `contracts/stencils#entries`.
  - Evidence: `contracts/stencils/stencils.go:450`, `:456`, `:597`.
- **F3 — LOW, consistency, card 11 (`11-pr-review-row-opens`).**
  - `internal/shedrecipe/doc.go:94` says "An auto-accept value, like the PRReview flag, is a bool". Card 11 removes the flag but rewrites only the doc's PRReview section (`doc.go:51-54`), so this line goes stale.
  - Evidence: `internal/shedrecipe/doc.go:94`.
- **F4 — LOW, consistency, card 12 (`12-retire-pr-review-keys`).**
  - When card 11 lands, these become stale until card 12 lands: the template comments at `loomengine/template.yaml:43` and `darnengine/template.yaml:12`, and the field docs at `loomengine/config.go:421` and `darnengine/config.go:60` ("the PR-Review stub's log line").
  - Card 12 covers all four, so this is noted only as an ordering detail, not a gap.
- **F5 — LOW, design, card 4 (`04-pr-review-config-keys`), unverified.**
  - A search for the sibling gate key `drift-verify` hits files card 4 does not list:
    - `internal/gatecli/doc.go`
    - `pattern/PATTERN-gate-slot.md`
    - `pattern/PATTERN-test-speed.md`
    - `docs/dotnet-hub.md`
    - `contracts/stencils/stencils.go`
    - `internal/loomshed/doc.go`
  - Any of them that lists the gate keys lyM reads also needs `pr-review`. I did not read their contents, so this is unverified.
  - Evidence: grep hit list for `drift-verify`.

**Tool calls**

1. Grep for `pr_review`, `pr_review_setting` and `PRReview` across the repo: the context lacked the full set of readers of the retired keys.
2. Grep for "stub", "spawns nothing", "opens, looks up and closes no pull request", and `childAtBeamAwait`: the context lacked the doc and comment sites the cards falsify.
3. Grep for `OutputFiles` outside the core shuttle files, plus `sweepOrphansOpportunistic` (`run.go:1040-1076`): the context lacked the other readers of an empty output list.
4. `run.go:600-622` and the `doneWhen` grep: the context lacked how `OutputFiles` reaches reed's `DoneWhen`.
5. Grep for readers of `shellguard.Rows` and `Forms`: the context lacked the no-denied-recovery scan's input.
6. Grep for files naming `drift-verify`: the context lacked the places that list the gate keys.
