# Decision record — test-suite-change-amplification

## Goal

A small production change costs edits in few test files, because test helpers and seam fakes live in one place instead of being copied per consuming package.
This is part 1 of the test-suite cleanup and carries the full scope of GitHub issues #324 (shared `internal/testkit`, one fake per cross-package seam) and #325 (git helpers in `gitkit`, `hubforge` as the only fabric fixture).
Both issues are closed and folded into this board task; their bodies, and #323's rules section, remain the file:line evidence, recorded against `fafb660c3` and to be re-verified against HEAD.
Part 2, `test-suite-derived-guards`, builds its invariant-scan harness under this task's `internal/testkit`.

## Scope

In scope, all of it:

- #324 decision section: reverse the "duplicate test helpers rather than share them" policy for cross-package seams, create `internal/testkit/`, land the Testkit Invariant in `CONSTRAINTS.md` with its enforcing scan.
- #324 §1: one shared `shuttleengine.Engine` fake and one `ReedOps` fake (`internal/testkit/shuttlefake`).
- #324 §2: one shared `Shuttle`, `BurlerRunner`, merge-shuttle fake, the webster placeholder seams, and the producer `Call` helpers (`internal/testkit/shedfake`).
- #324 §3: builders for `shedrecipe.Env`, `landingshed.Deps` and the loom recipe `Deps` (`internal/testkit/envkit`).
- #324 §4: one shuttleengine Runner/Run fixture inside `internal/shuttleengine`.
- #324 §5: one reflective "every seam filled" wiring test per CLI (`loomcli`, `battencli`).
- #324 §6: log capture, JSON envelope decoding, the shedadapters bouncer fixture, the webster git scratch helpers (folded into `gitkit`), `buildLyxBinary`, and `containsPath`/`equalStringSlices` replaced by `slices`.
- #325 §1: git query and commit helpers in `gitkit`, replacing the private copies.
- #325 §2: `hubforge` as the only fixture vocabulary in fabricengine tests.
- #325 §3: one scripted `fakeTmux` in `internal/reedengine` tests.
- #325 §4: exported sentinel errors in `internal/fabricengine` asserted with `errors.Is`.
- #325 §5: the listed duplicate-behaviour deletions.
- #325 §6: the reedcli reap-cycle table, the gitrepo oracle moved to `internal/gitrepo/internal/gitoracle`, one shared `*lyxcwd.Location` builder.
- #325 "Rules to adopt going forward": recorded in `CONSTRAINTS.md` and the fabricengine package doc.
- The board's operator directive "CONSTRAINTS.md may change": an audit of the invariants bearing on #324/#325's tests, with each one's disposition in D13 and the changes it implies (D4's single gitkit spawn rule).

Out of scope:

- #326, #327 and the tmux socket leak: `test-suite-derived-guards`.
- Invariants that bear only on #326/#327, listed in D13 as notes for `test-suite-derived-guards`; this task does not change them.
- The glyph-plan and file-tree fixture copies in `planglyph`, `loomshed` and `loomcli` (`glyphPlanFixture`, `glyphRepoPlanFixture`, `reworkParityFixture`, `writeGlyphRepoForCLITest`, `writeGlyphRepoForParityTest`, `writeGlyphRepoFixture`): #327 section 2's plan-fixture writer replaces them in `test-suite-derived-guards`; only their old-policy citations change here (D1).
- #325's "out of scope, noted for later" item (a force axis for the fabricengine `livestate_*` harness).
- Mutation testing (evaluated and not adopted in #323).
- Any change to production behaviour other than exporting the #325 §4 sentinels; operator-facing error text stays byte-identical.

## Decisions

### D1 — Shared kits live under `internal/testkit/<kit>/`

Decision: the "don't share test helpers" policy is reversed for fakes and builders of seams faked in two or more packages.
Each shared kit is its own package under `internal/testkit/<kit>/`, never under the package it fakes.
`internal/testkit` itself is a doc-only package (`doc.go`) stating the invariant, with the enforcing scan beside it.
A kit imports only the lowest packages that define the types it fakes, so in-package tests of a consumer can import it without a cycle.
A fixture used by one package stays in that package's `_test.go` files (`shuttleengine`'s Runner fixture, `reedengine`'s `fakeTmux`, `shedadapters`' bouncer fixture).
Where an import cycle forbids the kit (`shuttleengine`'s own tests, `shedrecipe`'s and `landingshed`'s own builders), that package keeps exactly one local copy.
Every comment citing the old policy is removed as its code is converted.
At the plan-fixture sites left to #327 (Scope), the citation is reworded without converting the code: the comment keeps its pointer to the copy's origin and drops the policy reference.

Rationale: per-package seam-enforcement scans walk their own directory recursively and skip only `_test.go` files, so a kit nested under a bound package would be scanned as that package's production code.
A separate top-level home keeps the scans unchanged.

Bound: the reversal lets a fake be shared across packages, so one wrong fake now misleads several packages at once.
It is bounded to seams faked in two or more packages, and each kit's fakes reproduce the contract of the copy it replaces (the richest existing copy wins, e.g. `shedadapters/singlellm_test.go`'s `Shuttle`).

### D2 — Kit names

Decision: `internal/testkit/shuttlefake`, `internal/testkit/shedfake`, `internal/testkit/envkit`, `internal/testkit/logcapture`, `internal/testkit/envelope` (the JSON envelope decoder #324 §6 placed at `internal/output/outputtest`), and `internal/testkit/lyxbin` (the shared `buildLyxBinary`).

Rationale: every shared test kit sits under one root that one invariant covers; an `outputtest` under `internal/output` would escape it.

### D3 — Testkit Invariant

Decision: a new `CONSTRAINTS.md` section, "Testkit Invariant":

- Packages under `internal/testkit/` are imported only from `_test.go` files.
- A kit imports no `internal/*cli` package.
- A kit spawns nothing: it imports neither `os/exec` nor `internal/gitexec`, and starts no process, tmux server or agent.
  Sole exception: `internal/testkit/lyxbin`, which runs only `go build` of `./cmd/lyx`.
- A kit holds fakes and builders only and asserts nothing about the package it fakes beyond `t.Fatalf` on its own setup failures and the `Call`/`RequireOutcome` helpers' outcome check.

Enforced by one AST scan in `internal/testkit`, modelled on the existing seam-enforcement scans, covering the importer rule, the `*cli` rule and the spawn-import rule with `lyxbin` named as the one exemption.
The scan obeys the Test Tier Purity Invariant; if it resolves the module root the way the `cmd/lyx` guards do, it gets an allowlist entry with a reason, like theirs.

Bound: the `lyxbin` exemption lets one kit spawn `go build`, and nothing else.
The token `lyxbin.` goes on the Tier Purity banned-token list, so only `integration`/`smoke`-tagged files can call into it.

Rationale: kits that spawn nothing are safe in Tier 1 tests; the `*cli` ban keeps kits below the wiring layer and out of import cycles.

### D4 — `gitkit` gains the git query and commit helpers

Decision: `gitkit` adds `Git` (trimmed stdout), `RevParse`, `CurrentBranch`, `BranchExists`, `IsAncestor`, `RevListCount`, `LsFiles`, `ExcludeLines`, `CommitFile` and `CommitFileOnBranch`, stdlib-only so the gitkit Leaf Invariant holds.
The private copies in fabricengine, gitrepo, fabriccli, cmd/lyx, hubforge, the webster pair, loomcli and landingshed convert to them.

The Test Tier Purity and Hermetic Git Test Environment scans stop listing gitkit helpers one by one.
Today `cmd/lyx/tierpurity_test.go`'s `bannedTokens` and `cmd/lyx/hermeticenv_test.go`'s `gitSpawnTokens` each keep their own list of which gitkit exports spawn git, and the two already disagree (`gitkit.MustRun` and `gitkit.SeedConfig` are git spawns to one scan and not the other, and `gitkit.GitStatusPorcelain` is in neither).
Both are restated as one rule: every `gitkit` export spawns git except `gitkit.HermeticGitEnv`.
One definition in `cmd/lyx`, shared by both scans, matches any `gitkit.` reference other than `gitkit.HermeticGitEnv`; neither scan names an individual gitkit helper any more.
The two `CONSTRAINTS.md` sections are reworded to state the rule, not the token list.
Converted helpers and existing untagged `gitkit` calls (in `internal/fabricengine/export_test.go`, `clone_test.go`, `internal/websterengine/config_test.go` at survey time) move to `integration`-tagged files; no new allowlist entry is added for them.

Bound: this tightens Tier Purity; untagged tests lose `gitkit.MustRun`, `SeedConfig` and `GitStatusPorcelain`, which spawn git today unflagged.
A future non-spawning gitkit export would be banned too until the shared definition exempts it by name.

Rationale: 84 private helpers, many the same one-liner, exist because external and internal test packages cannot reach each other's helpers; `gitkit` is reachable from both.
Ten new exports would otherwise mean twenty token-list edits across two scans, and every later gitkit helper two more.

### D5 — `hubforge` is the only hub fixture vocabulary

Decision: the fabricengine `fabricFixture` shim is deleted and tests use `*hubforge.Hub` directly, including `h.Topology` instead of hand-built `NewTopology(Config{})`.
`hubforge` adds `AddPairWith(tb, h, slug, opts)` and `OpenFabric(tb, h)`.
New fabricengine test files prefer the external `_test` package plus `export_test.go` accessors; this goes in the fabricengine package doc.

Rationale: one constructor-signature change today touches ~190 sites.

### D6 — Rules going forward land in `CONSTRAINTS.md`

Decision: the gitkit Leaf Invariant gains the clause "test git plumbing lives in `gitkit`; a package-local `_test.go` helper that shells out to git is a review flag", and the hubforge Fabric-Fixture Invariant gains "no test package wraps `hubforge` in its own fixture type".
Both are review discipline, stated as such in the section, not scans.

Rationale: without a stated rule the copies regrow; a scan over raw git spawns in test files belongs with part 2's scan harness.

### D7 — Fabricengine sentinel errors

Decision: `internal/fabricengine` exports `ErrInvalidSlug` (wrapping the specific reason), `ErrNoWarpBinding` and `ErrWarpBindingMismatch`; tests that ask "which refusal" use `errors.Is`.
Each operator-facing remedy string keeps exactly one test, in the engine; the CLI asserts once, generically, that the engine's message reaches the envelope unchanged.
Message text is unchanged.

Rationale: the same substrings are pinned in engine and CLI tests today, so a wording change breaks both.

### D8 — Reflective wiring test

Decision: `loomcli` and `battencli` each get one test that calls `wire()` and walks `shedrecipe.Env` with `reflect`, recursing into nested deps structs; every func, interface, pointer, map or slice-of-func field must be non-nil unless named in an `intentionallyNil` set.
It replaces the hand-picked nil-check tests; tests that assert values or identity stay.

Bound: `intentionallyNil` is an escape hatch that lets a nil seam through.
Each entry names one field path and carries a one-line reason; a field not listed fails the test.

Rationale: a new seam is checked without a new test, so forgetting it is no longer silent.

### D9 — Protected tests and assertions

Decision: #323's protected list holds: every `CONSTRAINTS.md` invariant scan, every regression test citing a finding, crucible round, sabotage proof or live observation, every `smoke`/`integration` live test, the gitrepo oracle, the fabricengine `livestate_*` harness and its selftests, and `internal/loomcli/wiring_commitstatus_integration_test.go`.
These are restructured at most, never deleted, and every assertion they make survives.
A scan whose scaffolding is touched keeps its allowlist, floor and token set verbatim, and is proved by planting one violation, watching it fail, and removing the plant.
`shedfake`'s `Call` helpers keep `Fatalf`-on-error semantics exactly.
Scripted fake answers keep their exact values; tests asserting call order use the fake's call log.

Rationale: the suite does not have too many checks; this task moves where they live.

### D10 — Test-comment and allowlist hygiene

Decision: when a test is touched, doc comments restating the test name are dropped, and no comment counts rows or names seams by list.
Touched allowlists use one entry per line without aligned values.

Rationale: #323's rules; both cut comment churn, which is a fifth of edited test lines.

### D11 — Docs

Decision: `internal/testkit/doc.go` and each kit's package doc describe the kit; `docs/overview.md`'s test-infrastructure passage on `gitkit` and `hubforge` gains `internal/testkit`.
No `manifest/designs/` doc: `testkit` is not a lyx CLI module.
`manifest/roadmap.md` is untouched.

Rationale: CLAUDE.md's docs-in-the-same-commit rule for cross-cutting infrastructure.

### D12 — Order of work

Decision: D1–D3 (testkit root, invariant, scan) land first, since the shared fakes depend on them.
#325's items are independent of `testkit` except the webster git helpers, which need D4's `gitkit` helpers first.
Conversion is package by package, so each step leaves the suite green.

Rationale: a red suite mid-task hides which conversion broke it.

### D13 — Invariant audit under the operator directive

Decision: each invariant bearing on #324/#325's tests gets one disposition; no invariant's intent is weakened.

Changed by this task:

- The do-not-share-test-helpers decision (recorded only in test comments, never in `CONSTRAINTS.md`): reversed and replaced by the Testkit Invariant (D1, D3).
- Test Tier Purity Invariant and Hermetic Git Test Environment Invariant: their two hand-kept gitkit token lists become one shared rule (D4); `lyxbin.` joins Tier Purity's tokens (D3).
- gitkit Leaf Invariant and hubforge Fabric-Fixture Invariant: each gains a review-discipline clause (D6); their import rules are unchanged.

Kept unchanged, with the reason:

- Hermetic Git Test Environment's per-package `TestMain` obligation: a one-time five-line file per git-spawning package that no ordinary change edits.
- Per-package seam-enforcement scans (Shed Producer-Seam, Treadle Runner-Seam, Shed Recipe Registry, Shed Verb-Set, Batten Bookend's proxy scan, loomrecipe's): their rules are what keep kits out of bound packages (D1); their copied scaffolding is #326's harness work.
- Completion Signal Invariant: its tripwire pins production return sites; the Runner fixture (#324 §4) restructures the tests it protects without touching the scan.
- Gate Self-Check Parity Invariant: the parity checks stay one per gate and this task leaves their fixtures in place; #327 section 2 shares them.
- Refusal Way-Forward Invariant: binds webster, shed and loom, not fabric, so D7's remedy-string rule does not touch it.
- Fabric Vocabulary Invariant: kits stay outside its owner set and must comply.
- Live-Substrate Spawn Observability: test-fixture machinery is already outside it, `lyxbin` included.

Notes for `test-suite-derived-guards`, not taken into scope here:

- The per-package `seam_enforcement_test.go` and `leaf_enforcement_test.go` copies share root lookup, walk, allowlist and floor scaffolding: #326's one-harness item, which can host this task's Testkit scan too.
- Tier Purity's and Hermetic Env's per-file allowlist entries whose only reason is resolving the module root through `go env GOMOD` mirror each guard's implementation; one shared root resolver in the harness retires them.
- CLI / Cobra Invariant help-tree tests, Sandbox Suite Coverage, the config-section registry, Cwd Resolution's constructor-anchoring and no-transients path mirrors, Shed Recipe Registry's row mirrors, Agent Name and Stencil Ownership tests: hand-kept copies of registries production owns, belonging to #326/#327.
- The glyph-plan and file-tree fixture copies named in `## Scope` stay for #327 section 2's plan-fixture writer; this task only rewords their old-policy citations.

Rationale: the operator authorized changing invariants that made the suite grow; only the gitkit token lists both bear on this task and force a hand-kept mirror, and the rest either do not amplify change or belong to part 2.

## Constraints

- `CONSTRAINTS.md` is authoritative; the new invariant and the D6 clauses land in it in the same commit as their code.
- gitkit Leaf Invariant: `gitkit` production code stays stdlib plus its listed packages.
- hubforge Fabric-Fixture Invariant: no package in `fabriccli`'s dependency set imports `hubforge`.
- Fabric Vocabulary Invariant: `internal/testkit/**` non-test `.go` files are production files to that scan and are outside its owner set, so they must not use the fabric-internal vocabulary.
- Test Tier Purity Invariant: kits spawn nothing (D3) and are usable from untagged tests; under D4's restated rule no untagged file calls `gitkit` beyond `HermeticGitEnv`.
- Hermetic Git Test Environment Invariant: any package whose tests reference `gitkit` beyond `HermeticGitEnv` has a `TestMain` calling `gitkit.HermeticGitEnv()` (D4).
- A changed invariant lands in `CONSTRAINTS.md` in the same commit as the scan and tests that enforce it.
- Per-package seam-enforcement scans stay unchanged; kits never sit under a bound package (D1).
- Semantic line breaks in every `.md` and multi-line comment.
- Build needs cgo; the integration and smoke tiers need git and tmux.

## Auto-mode assumptions

- No operator answered questions; every pick in `## Decisions` was made autonomously, guided by #323–#325.
- The surveyed duplication still exists at HEAD: spot checks confirmed the old-policy comments, 10 `AuditForksIncremental` fake implementations, 36 files using `newFabricFixture`, 82 `execHook` installations and the `buildLyxBinary`/`containsPath`/`equalStringSlices` copies.
  Exact file:line references drift; the plan recomputes them.
- `internal/output/outputtest` is replaced by `internal/testkit/envelope` (D2).
- The `lyxbin` spawn exemption is accepted to keep #324 §6's `buildLyxBinary` item in scope (D3).
- The operator directive on the board (changing `CONSTRAINTS.md` invariants that grew the suite) was relayed mid-session by the orchestrator and confirmed on the board; D13 is the audit it asks for.
- Tightening Tier Purity to all of `gitkit` (D4) is in the directive's spirit: it replaces two hand-kept lists, and the untagged files it newly flags spawn git in Tier 1 today.

## Open risks

- Size: #324 and #325 together are several thousand changed lines across many packages; webster batches must keep each conversion package-local so a failure localizes.
- Test-count accounting: the reflective wiring test (D8), the #325 §5 deletions and the §6 table conversion change the count on purpose; a miscounted baseline would mask an accidental loss.
- The `gitkit.` raw-substring rule (D4) trips on comments in untagged files that name a gitkit helper; those are reworded, as the scan's design already accepts.
- D4 may newly flag packages under the Hermetic Env scan (those reaching git only through `GitStatusPorcelain`); each gets a `TestMain`, not an allowlist entry.
- Part 2 will likely rebuild the Testkit scan on its harness; this task's scan must not assume its own shape is final.
- The `smoke` tier needs tmux and may be slow or flaky on the host; a flaky smoke result must be re-run, not waved through.

## Acceptance criteria

- `internal/testkit/` exists with `doc.go` and the kits named in D2; the Testkit Invariant is in `CONSTRAINTS.md`; its scan passes, and fails when one violation of each rule (a non-test importer, a `*cli` import, an `os/exec` import in a kit other than `lyxbin`) is planted, with each plant removed afterwards.
- No comment in the repo cites the "duplicate-test-helpers-rather-than-share-them" decision or says test-helper packages are not shared; at the plan-fixture sites left to #327 the code is unchanged and only the comment is reworded.
- No consumer package keeps its own `shuttleengine.Engine`, `ReedOps`, `Shuttle`, `BurlerRunner`, merge-shuttle or webster-placeholder fake, or its own full `shedrecipe.Env`/`landingshed.Deps`/loom `Deps` builder, except the cycle-forced cases named in D1.
- `shuttleengine` tests build Runners and Runs through one fixture; every Completion Signal row keeps its assertion.
- `loomcli` and `battencli` each have one reflective wiring test; adding a nil seam field to `shedrecipe.Env` without listing it in `intentionallyNil` fails that test (shown by a planted field, removed afterwards).
- `gitkit` exports the D4 helpers; the private git query/commit helpers they replace are gone; the webster pair uses them.
- `cmd/lyx`'s Tier Purity and Hermetic Env scans share one gitkit spawn definition and name no individual gitkit helper; planting an untagged `gitkit.RevParse` call fails Tier Purity, planting a `gitkit.GitStatusPorcelain` call in a package without the hermetic `TestMain` fails Hermetic Env, and `gitkit.HermeticGitEnv` in an untagged `TestMain` passes both; plants are removed afterwards.
- `CONSTRAINTS.md`'s Tier Purity and Hermetic Env sections state the D4 rule, and the gitkit Leaf and hubforge sections carry the D6 clauses.- fabricengine tests have no `fabricFixture`, use `h.Topology`, `hubforge.AddPairWith` and `hubforge.OpenFabric`.
- reedengine tests use one `fakeTmux`; the redundant `Width/Height` and `CollapsedRows/MinFullRows` resets are gone.
- The D7 sentinels exist and the "which refusal" tests use `errors.Is`; each remedy string is pinned in exactly one engine test.
- The #325 §5 deletions are made, and deleting the `validateWorktreeSlug` call in `Add` still makes a test fail.
- The gitrepo oracle lives in `internal/gitrepo/internal/gitoracle`, imports `gitexec` and never `gitrepo`, with an import-allowlist row enforcing it; `gogit_test.go`'s verbatim copy is gone.
- `docs/overview.md`, `CONSTRAINTS.md`, the fabricengine and kit package docs are updated in the commits that change their subject.
- Verification: `go test ./...`, `go test -tags integration ./...`, and the `smoke` tier of every package whose smoke files changed, all green before and after.
  Leaf test counts (top-level tests without subtests plus every subtest, from `go test -json`) are recorded before and after, and every drop is itemised against an intended deletion or consolidation.

## Notes for the plan writer

- Evidence lives in `gh issue view 323`, `324` and `325`; recompute every count and location against HEAD.
- `internal/loomshed` now exists beside `internal/loomrecipe`; locate the loom recipe `Deps` and its fixtures afresh.
- `internal/fabricengine/export_test.go`, `clone_test.go` and `internal/websterengine/config_test.go` are untagged and call gitkit helpers; D4 moves those calls to tagged files.
- `cmd/lyx/tierpurity_test.go` holds `bannedTokens` and its allowlist, `cmd/lyx/hermeticenv_test.go` holds `gitSpawnTokens`; `internal/shedverbs`, `internal/shedrecipe` and `internal/loomrecipe` hold the seam scans the Testkit scan is modelled on.
- `docs/overview.md` describes `gitkit` and `hubforge` near its test-infrastructure passage.
