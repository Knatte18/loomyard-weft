---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: cmd/lyx/gitkitspawn_test.go:1
    summary: New and changed Go comments across the range are fixed-column wrapped mid-sentence, breaking CONSTRAINTS.md's Comment Line-Break Convention
  - id: F2
    severity: LOW
    location: cmd/lyx/tierpurity_test.go:152
    summary: Card 3's edit to TestTierPurity_UntaggedTestsSpawnNothing left the condition as bad &&!pathAllowlisted, so the file is not gofmt-clean
  - id: F3
    severity: LOW
    location: internal/gitrepo/parity_test.go:185
    summary: Failure messages in parity_test.go and gogit_test.go still name the deleted oracleCurrentSHA, oracleChangedFilesSince and oracleCurrentBranch, so card 17's assert-no-callers grep still hits them
  - id: F4
    severity: LOW
    location: internal/gitkit/doc.go:1
    summary: gitkit's package doc still opens with git primitives only and a four-name export list, which card 2's Git and query helpers falsify
  - id: F5
    severity: LOW
    location: internal/hubforge/doc.go:3
    summary: hubforge's package doc calls its builder list its only exports, omitting TestShortname, SeedConfig and SeedFabricConfig
---

# Webster-Review round 1

Review range: `_lyx/shed/test-suite-change-amplification/status.json` names `product.parent` `main`, and no `_lyx/loom/rework/round-*` exists, so the subject is `git diff $(git merge-base main HEAD)..HEAD` (merge base `518dc7400`, 45 card commits).

Evidence gathered:

- `go build ./...`, `go vet -tags smoke ./...`, `go test ./...` and `go test -tags integration ./...` all pass at HEAD.
- `gofmt -l` over every added or modified `.go` file reports only `cmd/lyx/tierpurity_test.go` (F2).
- Test-count accounting, recomputed with the Shared Decision's own pipeline on the merge base (extracted with `git archive`) and on HEAD: 9457 before (matching card 1) and 9498 after (matching card 45).
  47 leaves disappear and 88 appear, so 9457 − 47 + 88 = 9498.
  Every removed leaf's top-level test is named in a commit body (cards 12, 13, 16, 35, 36), so the delta card 45 left unreconciled does reconcile.
  Every top-level `Test*` function removed from a directory is likewise named in some commit body.
- Card 11: every refusal's `Error()` text is byte-identical — `fmt.Errorf` without `%w` and `fmt.Sprintf` render the same, and `CloneHub`'s one-argument site keeps its exact format string.
  Each sentinel matches exactly the refusals the card assigns it.
  The only remaining refusal-text substring in a fabricengine test is in D9-protected `livestate_refusal_selftest_test.go`, which card 5 only restructured (its add/commit pair onto `gitkit.CommitFile`).
- Card 17: `gitoracle/oracle.go` imports stdlib and `gitexec` alone; its bodies differ from the deleted `oracle_test.go` only in exported names, `testing.TB` and the four `//gitexec:raw` markers; `checkedCallPinnedRawSites` gains exactly the one row; `gogit_test.go` and `parity_test.go` reach the oracle only through `gitoracle`.
  No code reference to the deleted symbols remains, but string references do (F3).
- Card 3: folding `gitkitSpawnReference` in after `firstBannedToken` keeps both allowlists, the skip dirs, the floor and the remaining tokens verbatim; `firstSpawnToken` falls through to the shared definition, and the shared definition is single.
- Kits: no kit's non-test file imports a `*cli` package or a spawn package other than `lyxbin`'s `os/exec`; assertions are `t.Fatalf` on setup plus the `shedfake` `Call`/`RequireOutcome` and `envelope` `RequireOK`/`RequireErr` pairs.
- Cards 35 and 36: the deleted and renamed wiring tests match the cards' lists, and the commit bodies record them.
- CONSTRAINTS.md additions and docs/overview.md match what the scans enforce and each traces to a card.

### [MEDIUM] Fixed-column wrapped comments in new and changed code

**Location:** cmd/lyx/gitkitspawn_test.go:1, and new or changed comments in cmd/lyx/hermeticenv_test.go, cmd/lyx/tierpurity_test.go, internal/gitkit/doc.go, internal/gitkit/gitkit.go, internal/gitkit/query.go, internal/fabricengine/sentinel.go, slug.go, warpbinding.go, internal/gitrepo/internal/gitoracle/oracle.go, internal/fabriccli/cli_test.go, internal/fabricengine/add_test.go and its moved `*_integration_test.go` headers, internal/reedengine (faketmux_test.go, attach_test.go, apply_test.go, generation_test.go, selvagepane_test.go, watchloop_test.go, watchdog_integration_test.go, ensuresession_integration_test.go), internal/reedcli (watchdog_test.go, spawnwatchdog_test.go), internal/shedadapters (bouncer_fixture_test.go, bouncer_seed_test.go), internal/shedrecipe (fixture_test.go, entries_bouncer_test.go), internal/loomrecipe (fixture_test.go, resume_test.go, shape_test.go), internal/battenrecipe/fixture_test.go, internal/landingshed/publish_test.go, internal/loomcli (wiring_test.go, smoke_test.go, smoke_attachprobe_test.go, smoke_gate_test.go), internal/shuttlecli/cli_test.go, internal/shuttleengine (fixture_test.go, startup_test.go), internal/websterengine (beginbatch_test.go, recoverbatch_test.go, strand_test.go).
**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires semantic line breaks — one sentence per line, further broken only at an independent-clause boundary — on every new or changed Go comment.
These comments, added in this range rather than moved verbatim, break mid-phrase at a fixed column (for example `shared by the Test Tier Purity` / `and Hermetic Git Test Environment scans` in gitkitspawn_test.go's header, `matches` / `ErrInvalidSlug.` in slug.go, `with %w` / `without changing that text.` in sentinel.go).
Moved-verbatim legacy comments are out of scope, and no line-width judgment is made.
**Fix:** Rejoin each mid-phrase break so every sentence sits on one line, keeping breaks only at sentence ends or independent-clause boundaries.

### [LOW] tierpurity_test.go is not gofmt-clean

**Location:** cmd/lyx/tierpurity_test.go:152
**Issue:** Card 3 changed `if bad && !pathAllowlisted(...)` to `if bad &&!pathAllowlisted(...)`; `gofmt -l` reports the file, the only changed Go file it reports.
**Fix:** Restore the space: `if bad && !pathAllowlisted(relPath, allowedSpawners) {`.

### [LOW] Failure messages name the deleted oracle functions

**Location:** internal/gitrepo/parity_test.go:185, 285, 288, 311, 355, 390, 413; internal/gitrepo/gogit_test.go:433, 450, 478
**Issue:** Card 17 deleted `oracleCurrentSHA`, `oracleChangedFilesSince` and `oracleCurrentBranch` in favour of `gitoracle.CurrentSHA`, `ChangedFilesSince` and `CurrentBranch`, but the `t.Fatalf` messages still print the old names.
A failing parity test then points at functions that no longer exist, and the Delete group's assert-no-callers grep (the plan-spec's "a grep for zero remaining callers") still finds ten hits.
**Fix:** Rename the messages to `gitoracle.CurrentSHA()`, `gitoracle.ChangedFilesSince()` and `gitoracle.CurrentBranch()`.

### [LOW] gitkit package doc keeps a stale export list

**Location:** internal/gitkit/doc.go:1
**Issue:** The doc opens "the below-fabric leaf holding git primitives only: MustRun, SeedConfig, HermeticGitEnv, and CopyRepo", and lines 11–13 repeat narrower lists ("MustRun, SeedConfig, and HermeticGitEnv are unpinned"; "gitkit hands out MustRun, SeedConfig, GitStatusPorcelain, HermeticGitEnv and ... CopyRepo").
Card 2 added `Git` and the query and commit helpers, and asked the doc to name them by group "rather than restating the export list"; the new paragraph does so, but the old lists now contradict it.
**Fix:** Replace the enumerations with group descriptions: the opening sentence says gitkit holds test git plumbing, and the pinning paragraph says every export but `CopyRepo` is unpinned.

### [LOW] hubforge package doc's "only exports" list is incomplete

**Location:** internal/hubforge/doc.go:3
**Issue:** The changed line says "its only exports are the Hub type, its geometry accessors, and the builders NewHub, AddPair, AddPairWith and OpenFabric".
The package also exports `TestShortname` (hub.go) and `SeedConfig`/`SeedFabricConfig` (seed.go), so the "only" claim is false.
**Fix:** Name the seeders and `TestShortname` alongside the builders, so the list matches the package.
