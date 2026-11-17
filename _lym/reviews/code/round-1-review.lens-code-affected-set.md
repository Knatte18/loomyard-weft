```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change in the diff, find everything outside the changed lines that reads, asserts or describes it, and report any of those the diff leaves wrong.

**Affected set per change**

- **`stencilstore.Pins`, `ChangedPins` and `readBoardCopy`.** Callers are `loomcli/stencilpins.go` and `pins_test.go`. `stencilstore/doc.go` covers them. The digest is `BodyHash`, the same strip-banner and LF normalisation that `lym stencil diff --all` applies to bodies. Covered.
- **`shedrun` record (`StencilPinsFile`, `StencilPinsRel`, `WriteStencilPins`, `ReadStencilPins`).**
  - Callers are `sharedbootstrap.go` step 2d and step 3, `warnChangedStencils` and the tests. `shedrun/doc.go` covers it.
  - The lock goes through `lock.TrackedFileLockPath`, which PATTERN-lym-files requires.
  - Reads and writes go through `internal/state`, which PATTERN-tolerant-records requires.
  - `StencilPinsRel` and `StencilPinsFile` share `runSegment`, so the legacy `self` fallback agrees between them.
  - `TreeRunRels` (`shedrun/paths.go:532-548`) does not list the file, but no tree reader needs it.
  - Run-directory deletion by `RunDirRel` takes the file along.
  - No other enumeration of the run directory checks its file names. A grep for `settle-records` found only the shedrun and sheddriver sources.
  - Covered.
- **`seedAndCommitBootstrap` writes and commits the pins.** Both `start` and `step` reach it. `loomcli/doc.go:194-196` describes it. The start help's "A run's first start pins" holds, since start always seeds. `step` also pins and never compares, which no help text claims otherwise. Covered.
- **`stencils_changed` envelope key on start and resume.**
  - The constant is `stencilsChangedField`.
  - Start help (`start.go:512-514`), resume help (`resume.go:251-253`) and `loomcli/doc.go:320-325` describe it.
  - Tests: `resume_test.go`'s `resumeEnvelope.StencilsChanged` and `bootstrap_test.go`. Start is unaffected on a first start, where nothing changes.
  - No spec under `_bolt/_lym/specs` or doc under `docs/` documents the start or resume envelope keys.
  - Covered.
- **Parent notice via `orchcli.NotifyParent` behind the `notifyParent` seam.**
  - Wired only in `newLoomCLI` (`cli.go:202-205`).
  - Test receivers built by `newTestSpawnAndWaitReceiver` leave it nil. That is safe today, because only a run with pins and a changed stencil calls it, and only the new test sets that up and also sets the seam.
  - Messaging goes through creel (PATTERN-agent-addressing).
  - The notice text points at `lym stencil diff <name>`. That command's behaviour is part of the set; see finding 1.
- **`docs/overview.md`.** It describes the verbs' seed commit and resume (lines 1146-1156 and 1294) without the pins or the notice. Nothing there is falsified, and no rule document requires it to land with this change. No finding.
- **Commit messages.** There is no `Pattern-Departure:` trailer, so none needs judging. See finding 2.
- **Declared PATTERN changes.** The directive declares none, so there is nothing to land.

**Findings**

1. **MEDIUM, consistency.** `internal/loomcli/stencilpins.go:66-68` (`stencilsChangedNotice`), echoed in `start.go:514`, `resume.go:253` and `loomcli/doc.go:323`.
   - Summary: the notice says "see each change with `lym stencil diff <name>`", but that command does not show the change since the run's pin.
   - Evidence: `internal/stencilcli/diff.go:3-8` and `runDiffOne` (`diff.go:~78-133`). `diff <name>` renders the base recovered by stamp against the currently shipped default. It shows only upstream changes not yet taken, falling back to shipped against on-disk.
   - Consequence: after a refresh that installed a new default, the board copy's stamp matches the shipped default, so the diff is empty even though the pin changed. A hand edit shows only through the fallback.
   - The Goal asks the notice to name `lym stencil diff`. The defect is the claim that it shows "each change". The wording should name it as the place to inspect each stencil's state, or say what it compares, rather than promise the run's delta.

2. **NIT, consistency.** Commit `e02ee4e9`'s message, against `internal/loomcli/stencilpins_integration_test.go:26-37`.
   - Summary: the message says it gives "the seed pins test a git-backed pair", but the test sets `FABRIC_SKIP_GIT=1`.
   - Evidence: the test's doc says "the bootstrap's commit needs no git repository". Only the records sibling is `git init`'d (lines 34-37). The pair itself is not git-backed.

Tool calls:
1. Grepped `RunTreeRels` construction, read `shedrun/paths.go:520-560`, and grepped readers of `startEnvelopeFields` and `stencils_changed` and strict decoders. The manifest lacked the run-directory enumerations and the envelope consumers.
2. Grepped the repo for `settle-records` and `stencil-pins`/`StencilPins` references. This checks for run-directory file lists and docs the manifest did not cover.
3. Grepped the specs, docs and contracts for loom resume and `stop_report` descriptions, and the notice topic tags. This looks for envelope docs outside the diff.
4. Grepped `docs/overview.md` and `docs/skills.md` for loom start, loom resume and `stencil diff` mentions. This checks for doc members of the affected set.
5. Grepped `CLAUDE.md`, `PATTERN.md` and `AGENTS.md` for an `overview.md` requirement and read `overview.md:1146-1156` and `1286-1302`. This settles whether the overview must land with the change.
