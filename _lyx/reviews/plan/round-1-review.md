---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/plan/31-tmuxkit-testmain-a.md
    summary: gitkit and gitrepo have untagged test files but integration-only TestMains, and card 31 gives neither a TestMain under the untagged set, so card 34's scan fails on the real tree
  - id: F2
    severity: MEDIUM
    location: _lyx/plan/12-per-cli-copies.md
    summary: card 12 misses loomcli's TestCommand_ReviewSubtree_ExactChildren (an exact-verb-set test) and shedcli's TestRunCLIIn_BareListingNeedsNoGitRepository (a NoArgs copy with a hand-kept verb list)
  - id: F3
    severity: MEDIUM
    location: _lyx/plan/22-stencilkit-loom.md
    summary: two hand-listed stencil seeders are outside cards 21-22 (loomcli/smoke_parentreview_test.go and shedadapters/bouncer_fixture_test.go), so card 21's smoke claim and card 22's repo-wide claim are both false
  - id: F4
    severity: MEDIUM
    location: _lyx/plan/28-statuscommit-seam.md
    summary: moving battencli's TestCommitStatusMessage into statuscommit leaves nothing pinning that battencli passes the batten prefix, and the Intent states no bound for that removed guard
  - id: F5
    severity: MEDIUM
    location: _lyx/plan/15-recipe-row-table.md
    summary: interruptpolicy_meta_test.go reads loomRowEngines, which card 15 folds into the one table, but the file is not a target
  - id: F6
    severity: MEDIUM
    location: _lyx/plan/23-claims-table.md
    summary: the claim shape has no section scope, yet websterengine's checks are section-scoped, so converting them weakens a must and strengthens a mustNot
  - id: F7
    severity: LOW
    location: _lyx/plan/25-claims-engines.md
    summary: card 25 targets anomalybody_test.go and shedadapters/rubric_test.go, which pin no stencil text, and promises moves from unnamed other files
  - id: F8
    severity: LOW
    location: _lyx/plan/18-plankit-webster.md
    summary: card 18 converts cliwire's seedPlanDir and webstercli's seedStandalonePlanDir, which write a placeholder rather than a plan, and lists template_test.go, which writes no plan
  - id: F9
    severity: LOW
    location: _lyx/plan/31-tmuxkit-testmain-a.md
    summary: cards 31-33 promise fixes to TMUX-reliant tests and environment-from-scratch spawns with no target, though HEAD has none of either
  - id: F10
    severity: LOW
    location: _lyx/plan/24-claims-webster.md
    summary: cards 24, 25 and 27 remove assertions without their Intent stating what the removal now lets through
  - id: F11
    severity: LOW
    location: _lyx/plan/17-plankit.md
    summary: D23 package docs for plankit, stencilkit and tmuxkit are not carried by their cards, and card 2 leaves docs/overview.md's Testkit scope sentence stale
  - id: F12
    severity: LOW
    location: _lyx/plan/30-tmuxkit.md
    summary: the 61-byte key bound is a hand-kept copy of reedengine's ServerName bound with nothing tying the two together
  - id: F13
    severity: NIT
    location: _lyx/plan/28-statuscommit-seam.md
    summary: card 28 does not say how battencli's marker write lands between the shared core's commit and push
---

### [BLOCKING] gitkit and gitrepo get no TestMain under the untagged set

**Location:** `_lyx/plan/31-tmuxkit-testmain-a.md` (targets `internal/gitkit/gitkit_test.go`, `internal/gitrepo/testmain_test.go`)

**Issue:** D18 requires a `TestMain` calling the entry point under every tag set that compiles the package's tests, and card 34's scan checks exactly that.
`internal/gitkit`'s only `TestMain` lives in `gitkit_test.go`, which is `//go:build integration`, but the package has untagged test files (`callerset_enforcement_test.go`, `leaf_enforcement_test.go`, `reexecguard_test.go`).
`internal/gitrepo`'s only `TestMain` lives in `testmain_test.go`, also `//go:build integration`, beside untagged `ancestry_test.go`, `keyvalidation_test.go` and `noforceadd_test.go`.
Card 31 edits both files but creates no sibling for either.
The card's own generic rule ("gains a `//go:build !integration` `testmain_test.go` sibling") cannot apply to `gitrepo`, because `testmain_test.go` is already the integration-tagged file.
After cards 31–32 the untagged set compiles both packages with no `TestMain`, so card 34's scan fails on the real tree and the suite goes red, which breaks D24's "each step leaves the suite green".

**Fix:** In card 31, create `internal/gitkit/testmain_test.go` as a `//go:build !integration` sibling calling `HermeticGitEnv()` then `tmuxkit.Main`.
For `gitrepo`, drop the `//go:build integration` line from `testmain_test.go` so its one `TestMain` compiles under every tag set; `gitkit.HermeticGitEnv()` sets environment only and is exempt from Tier Purity.
Update that file's doc comment, which justifies the tag.
Name both packages in the Intent.

### [MEDIUM] Card 12 misses two per-CLI copies

**Location:** `_lyx/plan/12-per-cli-copies.md`; evidence `internal/loomcli/cli_test.go:83-109`, `internal/shedcli/cli_test.go:30-44`

**Issue:** D5 deletes the per-CLI `NoArgs` copies and the exact-verb-set tests, and the acceptance criterion says no `*cli` package keeps one.
`loomcli`'s `TestCommand_ReviewSubtree_ExactChildren` pins the `review` group's children to the literal set `approve,delivered,notify,reject` and checks each `Short`: it is an exact-verb-set test.
`shedcli`'s `TestRunCLIIn_BareListingNeedsNoGitRepository` is a `NoArgs` copy carrying a hand-kept list of six subcommands.
Neither is named, and both names fall outside the patterns the card enumerates, so they survive the card.
Focus verdict: `configcli` and `stencilcli` hold no such copy, and every other package holding one is on the card's Edit list.

**Fix:** Name both tests in card 12's deletion list.

### [MEDIUM] Two hand-listed stencil seeders are outside cards 21–22

**Location:** `_lyx/plan/21-stencilkit-webster-orch.md`, `_lyx/plan/22-stencilkit-loom.md`; evidence `internal/loomcli/smoke_parentreview_test.go:91-104`, `internal/shedadapters/bouncer_fixture_test.go:106-112,165-180`

**Issue:** D11 converts every hand-listed seeder, and the acceptance criterion says no test hand-lists stencil byte slices to seed a directory.
`loomcli/smoke_parentreview_test.go` seeds `loom/` with `stencils.LoomTemplateParentReviewDelivery` and `stencils.LoomTemplateParentReviewBrief` by hand.
That falsifies card 21's "the `smoke`-tagged seeders already reconcile through the registry".
`shedadapters`' `newBouncerFixture` defaults to a hand-listed map of `stencils.BouncerTemplateSeed`/`BouncerTemplateJudge` and writes it through `writeBouncerStencils`, both in `bouncer_fixture_test.go`.
Card 22 names only `bouncer_seed_test.go`, whose maps flow into that fixture.
Card 22's own check ("no test in the repo hand-lists…") would therefore fail.

**Fix:** Add `internal/shedadapters/bouncer_fixture_test.go` to card 22 and convert its default seeding: seed all, then write the fixture rubric on top.
Add `internal/loomcli/smoke_parentreview_test.go` to card 22, with `go test -tags smoke ./internal/loomcli/` in its `**Verify:**` per D21.
Replace card 21's smoke sentence with an accurate one.

### [MEDIUM] Card 28 drops the only pin on battencli's commit prefix

**Location:** `_lyx/plan/28-statuscommit-seam.md`; evidence `internal/battencli/commitstatus.go:97-98`, `internal/battencli/commitstatus_test.go:182-205`, `internal/loomcli/wiring_commitstatus_integration_test.go:106`

**Issue:** The card turns each CLI's commit-message prefix into a parameter and moves `TestCommitStatusMessage` into `statuscommit_test.go`, where it can only test the renderer with whatever prefix the test passes.
`loomcli`'s `loom:` prefix stays pinned by the protected integration test's `loom: Discussion-Write -> running` subject check.
Nothing pins that `battencli` passes `batten`: its kept marker-skip tests count commits and never read the message.
The card claims "every commit message … reads as it does today" but removes the guard for that claim on one side and states no bound.

**Fix:** Have one of `battencli`'s kept marker-skip tests assert the message its fake `Commit` receives (`batten: <producer> -> <state>`), so the caller's prefix stays pinned in the layer `battencli` keeps.
State in the Intent that the log prefixes become caller-supplied and are unpinned, as they are today.

### [MEDIUM] Card 15 misses a reader of `loomRowEngines`

**Location:** `_lyx/plan/15-recipe-row-table.md`; evidence `internal/loomrecipe/interruptpolicy_meta_test.go:1-8,45-51`

**Issue:** The card moves `coverage_guard_test.go`'s `loomRowEngines` into `wantProducerTable`'s engine column.
`interruptpolicy_meta_test.go` ranges `loomRowEngines`, and its file comment explains its location by that variable.
The file is not a target, so the card either breaks the build or edits an unlisted file.
Focus verdict on the engine column: D9 has `coverage_guard_test.go` read "that one table instead of their own copies" and never be weakened.
The engine mapping is that guard's data, which `shedengine.ProducerDef` cannot supply, not a routing column, so keeping it in the one table is licensed.

**Fix:** Add `internal/loomrecipe/interruptpolicy_meta_test.go` to card 15's Edit list.
It should read the row-to-engine mapping from the one table and update its file comment.

### [MEDIUM] The claims table cannot express section-scoped checks

**Location:** `_lyx/plan/23-claims-table.md`, `_lyx/plan/24-claims-webster.md`; evidence `internal/websterengine/template_test.go:430-441,529-535`

**Issue:** Card 23's claim is a `must`/`mustNot` phrase checked against a stencil's whole raw text.
Several `websterengine` checks scope the phrase to one heading's section, for example `TestMasterTemplate_DoneCheckSectionTeachesLaterCardWarning` and `TestMasterTemplate_StatesPlanDriftRefusalEndsRunAsStuck`.
Converting one to a whole-file `must` no longer ties the phrase to its section, which weakens it.
A whole-file `mustNot` can fail on an unrelated mention elsewhere, which strengthens it.
D21 lets a protected assertion be restructured at most, never weakened.

**Fix:** Give card 23's claim row an optional section heading that limits the check to that heading's section, up to the next `## ` heading.
Have card 24 keep each section-scoped check scoped.

### [LOW] Card 25 targets files with nothing to move

**Location:** `_lyx/plan/25-claims-engines.md`; evidence `internal/loomengine/anomalybody.go:26-38`, `internal/shedadapters/rubric_test.go:31-151`

**Issue:** `anomalybody_test.go` asserts on `RenderAnomalyBody`, which builds its text in Go with no stencil.
`shedadapters/rubric_test.go` tests `ReadRubric`'s substitution and stamp stripping over fixture rubrics written by the test.
Neither holds a raw-stencil wording claim, so both are targets with nothing to move.
The Intent also commits to moving "a raw wording check in another test file of these three packages", which no target covers.

**Fix:** Drop both files from the targets and say why in the Intent.
Replace the open clause: a raw wording check found outside the listed files is reported, not moved silently.

### [LOW] Card 18 converts placeholder writers that write no plan

**Location:** `_lyx/plan/18-plankit-webster.md`; evidence `internal/cliwire/cliwire_test.go:83-93`, `internal/webstercli/wiring_test.go:88-99`

**Issue:** `cliwire`'s `seedPlanDir` and `webstercli`'s `seedStandalonePlanDir` write the bytes `placeholder` to `00-overview.md` only to satisfy `planDirHasContent`.
They spell no plan format, so "each converted fixture still produces an equivalent valid plan" cannot hold for them, and D10 does not reach them.
`websterengine/template_test.go` is listed but holds no plan writer.

**Fix:** Leave the two placeholder helpers as they are and say so.
Drop `internal/cliwire/cliwire_test.go`, `internal/webstercli/wiring_test.go` and `internal/websterengine/template_test.go` from card 18's targets.

### [LOW] Open-ended fixes in cards 31–33 have no targets

**Location:** `_lyx/plan/31-tmuxkit-testmain-a.md`, `_lyx/plan/32-tmuxkit-testmain-b.md`, `_lyx/plan/33-tmuxkit-socket-fixture.md`

**Issue:** Cards 31 and 32 promise to fix tests that rely on the caller's `TMUX`.
Card 33 promises to pass `TMUX_TMPDIR` through environment-from-scratch spawns it finds.
None of these edits has a target.
At HEAD no test file reads `TMUX` or `TMUX_PANE`, and every test-side `cmd.Env` appends to `os.Environ()`.
The ImpactSummaries of 31 and 32 ("no test body changes") hold only because the search finds nothing.

**Fix:** State the search result in each card, and turn the clause into a check: a hit is reported as a finding, not fixed outside the target list.

### [LOW] Cards 24, 25 and 27 do not state what their removals let through

**Location:** `_lyx/plan/24-claims-webster.md`, `_lyx/plan/25-claims-engines.md`, `_lyx/plan/27-shedverbs-duplicates.md`

**Issue:** Card 24 drops generic-wording claims and states the bound (provenance-citing claims are protected, drops itemised), but not what the drop now lets through.
Card 25 only defers to card 24.
Card 27 deletes `loomcli` tests and states the case-by-case bound, but not what `loomcli` stops checking.
Card 23 already states D12's let-through, which shows the shape the others need.

**Fix:** In cards 24 and 25, state D12's let-through: a dropped generic-wording phrase can change unnoticed, bounded to claims without cited provenance, each itemised.
In card 27, state D14's: `loomcli` no longer re-checks `shedverbs`' rendering and envelope itself, bounded by moving every case first and by the loom-specific tests that stay.

### [LOW] D23 docs are not fully carried

**Location:** `_lyx/plan/17-plankit.md`, `_lyx/plan/20-stencilkit.md`, `_lyx/plan/30-tmuxkit.md`, `_lyx/plan/02-testkit-scan-on-scankit.md`; evidence `docs/overview.md:573`

**Issue:** D23 requires each new kit's package doc to describe the kit.
Card 1 says so for `scankit`, but cards 17, 20 and 30 never mention a package doc.
Card 2 restates the Testkit scope in `CONSTRAINTS.md` and `doc.go`.
`docs/overview.md`'s sentence "`internal/testkit` holds the shared test kits, one package per kit, for seams faked in more than one package" keeps the old scope.

**Fix:** Add a package-doc sentence to cards 17, 20 and 30.
Have card 2 restate the overview sentence's scope per D19.

### [LOW] The 61-byte bound is an unguarded copy

**Location:** `_lyx/plan/30-tmuxkit.md`; evidence `internal/reedengine/server.go:73`, `internal/reedengine/server_test.go:74-86`

**Issue:** The longest key `reedengine.ServerName` produces is `maxSocketSafeBaseBytes` (48) plus `lyx-`, `-` and 8 hash bytes, which is 61.
`tmuxkit` repeats that number with nothing tying it to `reedengine`, so a later change to the cap leaves the kit's path-length check silently wrong.
This task's goal is to remove exactly this kind of copy.

**Fix:** Have `tmuxkit_test.go` assert that `reedengine.ServerName` of an over-long hub path fits the kit's bound.
`reedengine`'s production code does not import `tmuxkit`, so this creates no cycle.

### [NIT] Card 28 leaves the batten marker hook unnamed

**Location:** `_lyx/plan/28-statuscommit-seam.md`; evidence `internal/battencli/commitstatus.go:146-184`

**Issue:** `battencli` writes its marker after a successful commit and before the push.
The card puts commit and push in the shared core and says `battencli` keeps the marker write between them, but never says how the core exposes that point.

**Fix:** Name it: the core takes an optional after-commit callback that `battencli` uses for the marker write and `loomcli` leaves nil.

## Focus items settled with no finding

- Cards 3–10 against D2 and D4: every `*_enforcement_test.go`, every `go env GOMOD` exec, every root-lookup `exec.LookPath("go")` skip and every invariant-enforcing `runtime.Caller` root lookup is a target of some conversion card.
  `crosscompile_test.go`'s skip guards the build it runs, as card 4 says.
  The three files in `root-lookups-left-alone` pin the ly-drive skill and a deploy ldflag, and none of them enforces a `CONSTRAINTS.md` invariant.
- Card 19's repo-wide `format: 5` promise: every non-`planparser` test file spelling it is a target of card 18 or 19.
- Cards 31 and 32 together cover every package with an `integration`- or `smoke`-tagged test file, apart from the F1 gap.
  No edited `TestMain` file is `smoke`-tagged.
  No new `TestMain` lands in a git-spawning package: each such package passes the Hermetic Env scan today without one.
- Card 20's import-cycle note holds: `contracts/stencils` and `internal/stencilstore` reach only `proc`, `gitexec`, `lyxcwd`, `lyxdirs`, `logger` and `stencil`, and none of these is converted by card 21 or 22.
  `planparser` does not import `planglyph`, so card 19's `planglyph` conversion has no cycle either.
- Card 5's "CONSTRAINTS.md needs no edit" matches D20: neither invariant's text names an allowlist entry, and Tier Purity's `tmuxkit` rule lands in card 30.
- Card 15's engine-name column: settled under F5 as licensed.
