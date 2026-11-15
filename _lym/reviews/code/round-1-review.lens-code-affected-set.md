```yaml
lens: code-affected-set
outcome: reported
```

**code-affected-set:** for each change the diff makes, what reads or describes it outside the changed lines, and whether that set is covered.

## Per change: the affected set and whether it is covered

- **`RunCLI`/`RunCLIIn` removal.** Covered: `PATTERN-cli-cobra`, `PATTERN-shed-driver`, `docs/overview.md`, the `shedverbs`/`shortcutcli`/`shedcli`/`adoptcli` docs, the `gatecli` and `selfreportcli` comments, the `cwdmutation_test.go` reasons, every caller across the `*cli` tests, and the deleted `seamsignature_test.go`. `docs/research` and `docs/benchmarks` are historical, apart from F5.
- **`cmd/lym` `run(args, stdout, stderr)`.** Covered: every `cmd/lym` test call site and `docs/overview.md`.
- **logger span API.** Covered: the logger package doc and `docs/shared-libs/README.md`.
- **`dropSeedCommits` and `boardDropRequest`.** Covered: `destructiveguard_test.go`, `fabricengine/doc.go` and the `mutation.go` `KindCommitsDropped` comment.
- **`LoomReworkCoveragePath`.** Covered: `pathrules_test.go` and `config_test.go`.
- **`preflight.Wired`.** Covered: the preflight doc, the `webstercli/wiring.go` comment and the `stencilseed_integration_test.go` comments.
- **`WireJunctions` and `CodeLymLink`.** Covered: `fabricengine/doc.go`, `docs/shared-libs/lymcwd.md` and `hubforge/reanchor.go`. `docs/benchmarks/test-redundancy.md` is a report.
- **`PushAnchored` and `PushWarpAt`.** Covered: `fabricengine/doc.go`.
- **`snapshotWarpSHA`.** Not covered: F2.
- **`shedbuild.Load`.** Covered: the `shedbuild` doc and the `parse.go` header. Not covered: the contract spec, F1.
- **`AppendDecision`.** Covered: the `discussionparser` doc. Not covered: `append.go:228`, F3.
- **planglyph package wrappers.** Covered: the package doc, `plangate.go` and `index.go`. Not covered: `docs/overview.md`, F4.
- **Removals with every caller updated:**
  - `ReservedMarkers`: seatengine doc and `table_test.go`.
  - `agentconfig.Roles`: `config.go` comments.
  - `TerminalPhases`: `hibernatedaemon_test.go`.
  - `BareSiblingPath`: `weftname_test.go`.
  - `CleanupRetiredNotices`: orchengine doc and `state.go`.
  - `Launcher.Name`: `termwindow_test.go`.
  - `tmuxProcessName`: Windows-only test.
- **`NewRecordsStopError` moved to hubforge.** Covered: hubforge is a fabric-vocabulary owner (`internal/fabricvocab/fabricvocab.go:27`), so the move matches commit `ea8f1c36`'s message.
- **PATTERN.**
  - `PATTERN-test-only-code` rule line 11 says `PATTERN-test-isolation` bars gitkit and hubforge from a kit. That holds per `pattern/PATTERN-test-isolation.md:158-160`.
  - My directive declares no PATTERN changes, so there is nothing to check under "Declared PATTERN changes landed".
- **Commit messages.** Covered: no `Pattern-Departure:` trailers, and every message matches its diff.

## Findings

**F1 — MEDIUM, consistency, `contracts/specs/shed-recipe-spec.md:243` and `:255-256`**
- The repository's own spec still says `shedbuild.Load` "stays exported and covered as the entry a non-embedded consumer … would use". It also says the recipe is read "never `shedbuild.Load`".
- The diff deletes `Load` from `internal/shedbuild/parse.go` and deletes `internal/shedbuild/load_test.go`, so the spec now contradicts the code.
- Fix: rewrite the "Unused entry" paragraph and line 243 so they name `shedbuild.Parse` only.

**F2 — LOW, consistency, `internal/fabricengine/doc.go:386`, `:396`, `:418`, `:429`, `:480`, `:491`**
- The package doc still describes `Fabric.snapshotWarpSHA(tag string)` as the read half of the `Snapshot:` trailer mechanism, and says "A consumer reads the SHA via `snapshotWarpSHA`".
- That method is deleted along with `snapshot.go`. Its body now lives only in `SnapshotWarpSHAForTest` in `export_test.go`, a free function, and nothing in production reads the SHA.
- The diff added "Only tests call it" at line 387 but left the method name and the consumer claim.

**F3 — LOW, consistency, `internal/discussionparser/append.go:228`**
- `CheckAddedDecisionsKept`'s doc says "AppendDecision replaces the bullet".
- `AppendDecision` is now a test helper in `append_test.go`. The production writer is `AppendDecisions`.

**F4 — LOW, consistency, `docs/overview.md:768`**
- The doc says "`planglyph.ValidateFormat`/`Validate` call `planparser.ValidateFormat`/`Validate`".
- Those package-level functions now exist only in `internal/planglyph/planglyph_test.go`. Production goes through the unexported `validateFormat` and `validate` and the index methods.

**F5 — NIT, consistency, `docs/benchmarks/board-performance.md:93`**
- It says the CLI bench rows "drive `boardcli.RunCLI`" and points at `bench_cli_test.go`.
- This diff moved `bench_cli_test.go` onto `clihelp.Execute`.
- The file may be exempt as a measurement report.

## Tool calls
1. `git grep` of non-test files and docs for the deleted names (part one), to find references the reading manifest did not cover.
2. `git grep` for the remaining deleted names, `realClock`, `Roles` and the planglyph wrappers, for the same reason.
3. Read `contracts/specs/shed-recipe-spec.md:238-258`, `gatefixer/spawn.go:80-88`, the `websterengine/poll.go` header, `discussionparser/append.go:222-232`, and a docs grep for spans, to see each hit in context.
4. `git grep` of the `fabricvocab` owners, the `pattern/` kit import rules, and `git show` of `ea8f1c36`, to check the hubforge vocabulary ownership, the PATTERN rule's claim and that commit's message.
5. `git grep` for callers of `PollUntilTerminal` and for `gatefixer.Roles`, to see who used the deleted `realClock` and to rule out a false hit on "outside Roles". `gatefixer` has its own `Roles`.
6. `git grep` at the base commit for `PollUntilTerminal` and `realClock{}`. The live caller is in `recoverbatch.go`; call 5 only seemed to show none because its output was cut at six lines.
