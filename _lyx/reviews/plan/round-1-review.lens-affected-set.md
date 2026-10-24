```yaml
lens: affected-set
outcome: reported
```

Task: the affected-set lens. For each output a card changes, I traced its readers and checked which cards cover them. I found no missed reader that would break a test or a caller, and two partial-coverage gaps.

**Per change: the readers found and the cards that cover them**

- **`FailedTier` re-signed (card 2):**
  - Readers are `landingshed/verifygate.go:96` `recordFailure`, `gateslot/runlog_test.go:119` `testFailedTier`, and the `gateslot/doc.go:116` sentence.
  - Card 2 covers all three.
- **`FindTierTimeout` finding the build line (card 2):**
  - Readers are `verifytree/verify.go:477` `tierTimeoutDetail`, `verifytree/doc.go:13`, and the `TestWatchdogLine` and `TestVerify_Scenario` rows.
  - Card 2 covers all of them.
- **`RunRecord.Build` and the `-build-` file stem (card 2):**
  - Readers are `WriteRunRecord`, `gateslot/doc.go:111` (the record name), `report.go:159-160` (the report totals), `report_test.go:36-39`, and `summaryLogPathOf`.
  - Cards 2 and 11 cover them.
  - `ReadRunRecords` has no production caller outside `report.go`.
- **`LandingFailure.Build` (card 2):**
  - Readers are `verifygate.go`, `mergein.go`, `finalize.go`, `fixerrun.go`, the landing fixer stencil and the verifytree/landingshed tests.
  - The `recordFailure` path is covered by card 2. I did not inspect `fixerrun.go`'s use of the record, because I hit the tool budget.
- **The timed-out reason text in `verifyGate.verdict` (card 2):** only `verifygate.go` and `verifygate_test.go` read it, and card 2 covers both.
- **`tierFingerprint` re-signed (card 9):**
  - Readers are `scope.go:245` `diffBaseGroups`, `green.go:285` `subtract`, `green.go:342` `recordPass`, and `green_test.go:35,46,96,241` inside `TestGreenRecord` and `TestGreenRecord_NewestWholeAncestor`.
  - Card 9 covers all of them.
- **`openGreen` re-signed (card 9):** the one caller is `run.go:70` `runTest`, covered by card 9. The test literals (`green_integration_test.go:40`, `green_test.go:116`) build `greenRun` directly and keep compiling with the new fields.
- **`renderTierCommand` re-signed (card 7):** readers are `tierrun.go:136` `runInvocations` and `tiers_test.go:429` `TestRenderTierCommand`. Card 7 covers both.
- **The scoper refusal text `want go or none` (card 1):** readers are `config.go:211` and the `config_test.go:148` "unknown scoper" row in `TestLoadConfig`. Card 1 edits `TestLoadConfig`, so this is covered.
- **The `--guards` refusal reworded per scoper (card 8):** the reader is the `tiers_test.go:347` row, which asserts only the substring `scoper is "none"`. That still holds, and card 8 adds the `dotnet` row.
- **The empty-scope "takes no slot" sentences:**
  - Readers are `gatecli/doc.go:59` and `run.go:347` (the `runListed` doc).
  - Cards 9 and 11 rewrite `doc.go`, and card 11 edits `runListed`.
- **The gate summary in `docs/overview.md:461`:** card 12 covers it.
- **The module table:** card 6 covers it.
- **The scoper's doc comment and the `lyx gate test` help text:** these are the two partial gaps below.

**Findings**

1. **LOW · consistency · card 1 (`01-gate-config-keys.md`).**
   - Summary: card 1 adds `dotnet` to the scoper but does not name the `Config.Scoper` field comment, which still says "go or none".
   - Card 1 rewrites the scoper refusal, `doc.go` and `template.yaml`. Its Intent names `Tier.Command`'s comment for the new placeholder, so the parallel `Config.Scoper` comment is the one member of the set it leaves out.
   - Evidence: `internal/gateslot/config.go:34`, `// Scoper is how a computed, base or card scope maps to targets: go or none.`
   - Fix: add a clause to card 1's Intent that `Config.Scoper`'s comment names `dotnet`.
2. **LOW · consistency · cards 8 and 11 (`08-dotnet-scoper.md`, `11-shared-build-step.md`).**
   - Summary: `lyx gate test`'s help text is part of the observable-behaviour set these cards change, and no card updates it.
   - The help says each tier prints a summary line and writes its record, but card 11 adds a build line and record. It describes the guard step only "under the go scoper", while card 8 adds `dotnet`'s `--guards` refusal.
   - Every package doc is updated, but `reportCmd`'s help is the only help text any card edits.
   - Evidence: `internal/gatecli/cli.go:55-72`, the `testCmd` Long text, "Each tier prints a summary line to stderr and writes its record and log under the gate directory" and "--guards, and a narrower scope under the go scoper, also run the //lyx:guard tests outside the scope."
   - Fix: card 11 adds `cli.go`'s test command Long text to its Edit targets, naming the build step's line and record. Card 8 adds a clause that `--guards` is refused under `none` and `dotnet`.

Tool calls:
1. Grep for callers of `tierFingerprint`, `openGreen`, `renderTierCommand` and `FailedTier`: the manifest held their definitions, not every call site.
2. Grep for the green-test literals and for readers of the changed strings (scoper refusal, record stem, report fields, "takes no slot"): needed to find the assertion sites.
3. Grep for `ReadRunRecords` callers, the `--guards` refusal tests and outside readers of the summary line: needed to find consumers outside the manifest.
4. Grep for files naming `{parallel}` or the scoper: needed to find docs and help that list placeholders.
5. Read of `cli.go:55-80` and the `Config.Scoper` comment: the help text was not in the manifest.
6. Grep for `LandingFailure` readers, the `tiers.tier<N>={...}` assertions and the scoper in `docs/overview.md`: needed to find the reader set of the record and of the reason text.
