```yaml
lens: code-affected-set
outcome: reported
```

The affected set of each change: the callers, docs and config the diff touches. Two members are left wrong and one doc is stale.

**Per change, the set found, covered or not**

- **`redundantCmd(goBinary, minuteUnit, grace)` signature**: the only caller is `internal/gatecli/cli.go:136`. Covered.
- **`runRedundant(ctx, out, redundantRequest)` signature**: the only caller is `redundant.go:113`. The tests go through `Command()`. Covered.
- **Verb behaviour, now slot-held with builds and runs, and its help text**:
  - `docs/overview.md` (gatecli line and verb paragraph): covered.
  - `internal/gatecli/doc.go`: covered.
  - `internal/gateslot/doc.go` (non-tier holders list): covered.
  - `pattern/PATTERN-test-speed.md` (runs list and the `lym gate redundant` bullet): covered.
  - `PATTERN.md:54` index line: nothing in it is stale. Covered.
  - `internal/gatecli/testmap.go:83,300` mention the verb only as a reader of the maps. Still true. Covered.
  - `README.md:78` ("lists from them the tests of a package that other tests already cover"): not updated (finding 3).
- **`redundancy_build_timeout_min` and `redundancy_test_timeout_sec` now also bound `lym gate redundant`**:
  - `internal/gatecli/redundant.go` help and `doc.go`: covered.
  - `internal/gateslot/doc.go:172-176` and `internal/gateslot/template.yaml:4-5`: not updated (finding 2).
- **Output keys** (new `refuted`, `mutation`, `mutants_killed`; `candidates` narrowed): the only reader is `redundant_integration_test.go`. Docs `doc.go`, `overview.md` and the pattern are covered. The `covering` field of `candidates` and `refuted` is left inconsistent with the verdict (finding 1).
- **New refusal for a stale map** (other Go files): reached by the `RefusesAMapBuiltFromOtherFiles` row, which asserts its message, as PATTERN-refusal-way-forward asks. Covered.
- **`Toolchain.RunAlone` with profile `""`, `runAloneArgs`**: existing callers in testmap and the redundancy step pass a non-empty profile, so their behaviour is unchanged. Tests are covered. `runner.go:1-2` and `RunResult`'s doc still say "coverage binary". This is cosmetic and not filed.
- **New exported `testcover.CoveringTests`, `PlanMutants`, `JudgeMutants`, `ScanSourceReaders`, `ProfileBlocks`, `BuildMutantBinary`**: each is called from `gatecli` production code, so each is reachable from `main` (PATTERN-test-only-code). The `testcover` doc is updated. Covered.
- **`testmap_integration_test.go` fixture cut to one test**: the change is local, and the comment states why. Covered.
- **Declared PATTERN changes**: the directive declares none, so there is nothing to check.
- **Commit messages and `Pattern-Departure:` trailers**: no commit carries a trailer. The figure in `4b066972` ("14.5 minutes") is superseded by the re-run in `e6a52ffc`. Commit history is not fixable through the diff, so this is not filed.

**Findings**

1. **MEDIUM · consistency** — `internal/gatecli/redundant.go:376-392`
   - The `covering` field printed for `candidates` and `refuted` is not the set the mutation verdict judged against.
   - Both rows print `verdict.Covering`, the coverage pass's list. For a removable test that list is drawn from the tests the greedy pass kept, and it can include source-check tests.
   - The verdict instead judges against `testcover.CoveringTests` over `covers` (`redundantmutants.go:131-139`), which leaves out source checks and ignores the greedy pass.
   - So a `refuted` row can list `unique_mutants` "that none of its covering tests killed" (`redundant.go:172-173`) beside a `covering` list naming other tests, and readers of the output are misled.
   - The docs say the covering tests are the `CoveringTests` pick: `doc.go:1138` and the summary's "Covering tests are the set cover…".
   - Evidence: `redundant.go:383-387`; `redundantmutants.go:131-139`; `verdict.go:110-116`.

2. **LOW · consistency** — `internal/gateslot/template.yaml:4-5` and `internal/gateslot/doc.go:172-176`
   - The config key comments still say these bounds apply to "a coverage map run or the redundancy report step".
   - The test bound's comment says a test past it "is left out". Under `lym gate redundant` the same keys now bound the mutation check's builds and runs, and a timed-out run counts as a kill.
   - The diff updates the verb's own docs (`redundant.go:102`, `gatecli/doc.go:1146`) but not the key's own documentation.
   - Evidence: `template.yaml:4-5`; `gateslot/doc.go:172-176`.

3. **LOW · consistency** — `README.md:77-78`
   - It still describes `lym gate redundant` as listing "the tests of a package that other tests already cover", the coverage-only definition the task fixes.
   - `docs/overview.md` was updated to "confirmed by mutation", but the README was not.
   - Evidence: `README.md:78`.

**Tool calls**

1. Grepped for docs referencing `gate redundant` and the two timeout keys: the manifest held no search of docs, the README or config outside the diff.
2. Read `README.md`, `testmap.go`, `gateslot/doc.go:165-180` and `PATTERN.md`: they reference the verb and the keys, and the manifest lacked their text.
3. Read the `README.md:74-80` context, grepped for the old `Short` text, and read the `template.yaml` header: to confirm the README wording and the template key comments.
