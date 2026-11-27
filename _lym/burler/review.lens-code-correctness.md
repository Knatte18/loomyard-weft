```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, running each added or changed function and branch against concrete inputs.

**List (function or branch: case run, result)**

- `passagesOf`: shared, single-engine, doubly aimed and unreached entries; correct per plan.
- `Renderer.FileEntries`: unknown or empty engine gives nil, a read error is returned; correct.
- `cmd/lym` reader closure: a `Passages` or `FileEntries` error gives the zero `Standing` and a Warn; correct.
- `StandingPassages`: empty board, no reader, reader error; correct.
- `MissingItems`, `Omitted`, `WriteManifest`, `ReadManifest`: empty manifest, engine-keyed and settings items, omitted item counted present, absent file; correct.
- `copyManifest`: no record, undecodable record, blocked `manifests` path; each is non-fatal and correct.
- `sweepOrphans`, `findRunByStrand`: `manifests` skipped; correct. The other run-root scans I read (activity, attach, session read, step stop, engines run) skip a directory with no `run.json`.
- `Claude.Prepare` and `writeLaunchManifest`: read-back, anchor resolve, no repository, `dotgit` error; correct.
- `nativeManifestItems`: no repository, working directory equal to root, correct. Empty instruction file, see F1. User setup warnings, see F2.
- `claudeManifest` and `guardCandidates`: guards on and off (`claudeDenies` holds only the fork and question denies, so the two-way key choice is right), nudge off, empty notice; correct.
- `skillListing`, `UserSetup`, `pluginSessionStartContexts`: no skills, nil runner, runner error; correct.
- `Codex.Prepare` and `codexManifest`: quoting and substring checks, nudge off, unresolved skill, pointer-only and self-read `CLAUDE.md`, marked and unmarked override; correct. Empty instruction file, see F1.
- `claudeFileCandidates`: working directory equal to root and nested; correct.
- `resolveUserSetup`: unrun reason is silent, any other reason warns; correct.
- `build`, `New`, `NewWithSeams`, nil `RunHook` fallback; correct.
- `ConfiguredSandbox`, `UnrunHooks`, `PromptItems`, `Unmet`: correct.
- `PrepareEach`: role with no skill list, `Prepare` error, no record; correct.
- `Compare`: set with no compared engine, engine with and without forks; correct.
- `Report`, `report`, `LookupSeat`: temp directory removed on every path, unknown name, failed builder; correct. Real burler seat names, see F3.
- `seatsOf`, `seatSpecs`, `seatengine.Specs`: model pins cleared, table not mutated; correct.
- `FixReviewSeatTable` and the catalog builders: absent gate config, no fan; correct.
- `manifestCmd`, `manifestFields`, `worktreeManifestReport`: report error, nothing missing, items missing, no unmet requirements; correct.

**Findings**

**F1. MEDIUM, class `design`**
- Location: `internal/shuttleengine/codexengine/manifest.go`, the `instructionFiles` loop in `codexManifest`, together with `claudeengine/manifest.go` `nativeManifestItems`.
- Summary: an empty user instruction file makes parity report a false missing item.
- Case: `~/.claude/CLAUDE.md`, or a `CLAUDE.md` above the repository root, is a 0-byte file.
  - `instructionFiles` returns it with `found=true` and `Text=""`.
  - Claude's `nativeManifestItems` lists every item of `setup.InstructionFiles` as `instruction-source/<abs path>`.
  - Codex lists it only if `inSetting(file.Text)`, and `inSetting("")` returns false. `developerInstructions` skips empty items too.
  - `Compare` then returns the item as missing.
- Effect: `lym shuttle manifest` exits non-zero for every seat on that machine. Its way forward ("give the Codex engine a delivery") cannot be followed, because there is nothing to deliver.
- Fix direction: Claude should not list an instruction file whose text is empty.
- Evidence: `claudeengine/usersetup.go` `instructionFiles` and `readInstructionFile`; `claudeimports.go:28-39`; `codexengine/manifest.go` `inSetting` and the `in.instructionFiles` loop.

**F2. LOW, class `design`**
- Location: `internal/shuttleengine/claudeengine/manifest.go`, `nativeManifestItems`.
- Summary: `setup.Warnings` is dropped without being logged.
- Case: `installed_plugins.json` is malformed, or a `SKILL.md` is unreadable.
  - `UserSetup` returns a nil error and puts the cause in `Warnings`.
  - `nativeManifestItems` logs only `err`.
- Effect: the Claude manifest silently lacks `skill-listing` and `plugin-context` items. Since Claude is the reference, the comparison then asks nothing of Codex for them, and a Claude run record gives no sign of the gap. The Codex engine logs each of these warnings in `resolveUserSetup`.
- Evidence: `claudeengine/manifest.go` `nativeManifestItems`; `claudeengine/usersetup.go` `UserSetup` and `installedPluginRoots`.

**F3. MEDIUM, class `design`**
- Location: `internal/shuttleparity/tables.go`, `burlerSeatTables`.
- Summary: the burler seats are cataloged under made-up strand roles, so `lym shuttle manifest` refuses the roles real runs use.
- Case: `lym shuttle manifest burler-review`, the chair role of a profile with an empty `RolePrefix` (`ReviewRolePrefix`, `ledRole`).
  - The catalog builds only the prefixes `solo` and `fanned`, giving names such as `solo-burler-review`.
  - `LookupSeat` therefore answers "no seat is called".
- Effect: D10 takes a seat by its strand role, and D11's bench rule runs the verb "for the same seat" as a run's record. A burler role copied from a record does not resolve.
- Related, from the same function: both calls set `Assist`, so the `…burler-review` chair is always the assisted variant. The unassisted reviewer is never built.
- Note for judging: plan card 6 asks for "different stand-in `RolePrefix` values" so that names do not collide. One empty prefix would meet that too.
- Evidence: `shuttleparity/tables.go` `burlerSeatTables`; `burlerengine/review.go:25-31,73`; `burlerengine/assist.go` `withAssistant`.

**Tool calls**
1. The run-root scans in `activity`, `attach`, `sessionread`, `stepstop` and `enginesrun`: the context held only the first lines of each, not what they do with a record-less directory.
2. `claudeDenies` and `creelFlagDirForHook`: needed the full deny table for the guard key choice, and to check that `Prepare` writes nothing outside the run directory.
3. `RolePrefix` sites, `ReviewChairRole` and `claudeimports.Expand`: the real seat names and the result for an empty file were not in context.
4. The role constants and `ledRole`: needed to resolve the actual plan and review strand names.
