---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/02-pattern-warp-root.md
    summary: Card 2 passes burler's and webster's WorktreeRoot to Directive, but in hub mode that field is the anchor path, so a subpath-anchored hub never finds PATTERN.md at the repo root
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/02-pattern-warp-root.md
    summary: Card 2 Intent makes a whitespace-only overview inactive without stating what bounds that skip
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/02-pattern-warp-root.md
    summary: Comments and docs that place PATTERN under _lyx or describe the old pattern leaf are covered by no card
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/14-vocab-sweep-tests-core.md
    summary: Cards 14 to 16 list test files whose only weft or warp hits are vocab-exempt identifiers, so nothing in them needs rewording
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/plan/18-vocab-scan-extend.md
    summary: Card 18 does not say how the scan treats internal/configsync test files or whether its sandbox skip reaches tools/sandbox
  - id: F6
    severity: NIT
    class: decision
    location: _lyx/plan/20-spawn-role-constants.md
    summary: Card 20 leaves the reed spec type its literal-role scan covers as whatever struct takes a Role, instead of naming it
---

Review of the plan against `_lyx/discussion/decision-record.md`.
Every Decision D1–D14, the Constraints and the Operator additions are carried by a card.
No card adds scope the record does not license, apart from the no-op file entries in F4.
Checked mechanically: the entry-map table has one row per `CONSTRAINTS.md` section after the merges the record allows, and the 43 `bg` rows are exactly the background files card 5 creates.
Every tracked file that names `CONSTRAINTS` is listed by one of cards 7–10, 12 or 13.
Every non-owner file the extended scan would flag is in cards 14–18, apart from F5's `configsync` case.
Every literal `Role:` at a spawn site is in card 20's list.
`verify:` runs `go test ./...` and `go test -tags integration ./...`, so every package a card targets has its tests run, and the smoke-tagged files compile through `go vet -tags smoke`.
No `Custom` card exists.
Cards 18, 23 and 4 state what they let through and what bounds it; card 2 does not (F2).
The Edit and Delete `ImpactSummary` fields are real conclusions.

### [MEDIUM] Hub-mode burler and webster geometry names the anchor path, not the repo root

**Location:** `_lyx/plan/02-pattern-warp-root.md` (call-site list); Shared Decision `directive-root` in `_lyx/plan/00-overview.md`
**Issue:** The plan says `burlerengine` passes `e.geom.WorktreeRoot`, and `RenderMasterPrompt` and `RenderRecoveryPrompt` pass their worktree-root parameter.
The `directive-root` decision says every call site passes "its spawn's worktree root", and the record wants `PATTERN.md` resolved from the warp worktree root.
In hub mode those fields are the anchor path.
`internal/hubgeom/hubgeom.go` (`BurlerGeometry`) and `internal/hubgeom/webstergeom.go` set `WorktreeRoot` to `l.AnchorPath()`, never `l.WorktreePath()`, and their comments say the two diverge in a subpath-anchored hub.
Under a subpath anchor the directive would read `<anchor>/PATTERN.md`, find nothing and render no directive, with no error.
Card 2's only subpath test covers `loomengine`, so nothing would catch this.
Two smaller points in the same card:
- `RenderMasterPrompt` takes both `worktreeRoot` and `anchorRoot`, and after the change `anchorRoot` would be unused.
  The card says to drop it only "where it fed only the directive", without saying it applies here.
  That change also needs the call in `runlevel.go` and the test calls in `template_test.go`.
- `RenderRecoveryPrompt` already has a `promptWorktreeRoot` parameter, so "replaces its `anchorRoot` parameter with the worktree root" is ambiguous about which value it keeps.
**Fix:** Have card 2 give burler and webster a told repo-root value that is the real worktree root in hub mode and the target in standalone mode, rather than reuse the anchor-valued `WorktreeRoot`.
Fill it in `hubgeom`'s and `standalonegeom`'s burler and webster geometry constructors, and add those files to card 2's Edit list.
Add a subpath-anchored case for burler and webster, as `loomengine` has.
State that `RenderMasterPrompt` drops `anchorRoot`, and which parameter `RenderRecoveryPrompt` keeps.
Update the `directive-root` decision to match.

### [LOW] Card 2 gives no bound for the whitespace-only skip

**Location:** `_lyx/plan/02-pattern-warp-root.md` Intent
**Issue:** The card makes a whitespace-only `PATTERN.md` yield no directive, so an emptied overview now disables injection silently.
The rubric's attack-surface rule wants the Intent to state what the change skips and what bounds it.
D4 states the bound: only whitespace-only content is inactive, any other content is inlined, and card 6's checker reports an empty overview and one with no entry bullet.
Card 2 states none of it.
**Fix:** Add a sentence to card 2's Intent giving the bound.
Only whitespace-only content is inactive, any other content is inlined verbatim, and card 6's real-file test fails an emptied loomyard overview.

### [LOW] Stale `_lyx/PATTERN.md` and old-leaf descriptions are covered by no card

**Location:** `internal/fabricengine/junction.go:189`, `internal/fabricengine/unwire.go:30`, `tools/sandbox/SANDBOX-FABRIC-SUITE.md` (the `_lyx/PATTERN.md` lines), `docs/overview.md` (module tree row for `internal/pattern`, and the shared-infrastructure sentence)
**Issue:** After this task PATTERN lives at the repo root.
Two fabric comments still say `_lyx/PATTERN.md` is preserved or protected.
The sandbox fabric suite doc still says `lyx fabric pull` reports `_lyx/PATTERN.md`-touching commits, which is the residue report card 1 removes.
Card 1's grep keys (`PatternResidue`, `pattern_residue`, `PATTERN-residue`) do not match that wording.
`docs/overview.md` describes `internal/pattern` as computing whether `_lyx/PATTERN.md` is present and feeding code-touching agents, which is wrong after cards 2–4.
CLAUDE.md requires `docs/overview.md` to change in the same commit when the module table changes.
No card names these files for this change.
**Fix:** Add the two fabric comments to card 2's Edit list, as it already edits the neighbouring `unwire_test.go` comment.
Add `tools/sandbox/SANDBOX-FABRIC-SUITE.md` to card 1.
Add the `docs/overview.md` row and sentence to card 13, or to card 2, rewritten to the new rule and the five roles.
Extend card 1's residue grep with the `_lyx/PATTERN` spelling.

### [LOW] Sweep cards list files that need no rewording

**Location:** `_lyx/plan/14-vocab-sweep-tests-core.md`, `_lyx/plan/15-vocab-sweep-tests-loom.md`, `_lyx/plan/16-vocab-sweep-tests-rest.md`
**Issue:** After stripping the `vocab-exempt` tokens, these listed files carry no `weft`/`warp` hit, so the sweep has nothing to change in them.
Their only hits are exempt identifiers such as `PairWarpWorktree`, `PrimeWeft` and `WEFT_SKIP_GIT`:
- `cmd/lyx/stencilseed_integration_test.go`
- `internal/loomcli/circling_integration_test.go`
- `internal/loomcli/integration_driverbootstrap_test.go`
- `internal/loomcli/landingdeps_integration_test.go`
- `internal/loomcli/smoke_driverstrand_test.go`
- `internal/loomcli/smoke_drivingsurface_test.go`
- `internal/pairteardown/teardown_integration_test.go`
- `internal/reedcli/list_integration_test.go`
- `internal/reedcli/naming_integration_test.go`
- `internal/reedcli/watchdog_integration_test.go`
- `internal/reedcli/watchdogreap_integration_test.go`
- `internal/shedcli/seed_integration_test.go`
- `internal/webstercli/acceptaudit_test.go`
- `internal/webstercli/restoreplan_test.go`

The record licenses rewording existing hits, not touching clean files.
Listing a no-op target invites a rename the `vocab-exempt` rule forbids.
**Fix:** Remove those files from cards 14–16, keeping every file that still has a hit after stripping.

### [LOW] Card 18 leaves two scan boundaries undecided

**Location:** `_lyx/plan/18-vocab-scan-extend.md`
**Issue:** `internal/configsync` is an owner directory with a narrow carve-out, string literals and comments but not identifiers.
Card 18 extends the scan to every test file "outside the owner set".
It does not say whether `internal/configsync/configsync_test.go` is in scope, and it uses `warpPath` as an identifier at about line 507, so the choice decides whether the scan passes.
Card 18 also skips "`sandbox/` fixture directories", while cards 8 and 14 sweep `tools/sandbox/*_test.go`, which have many hits.
The skip must therefore match the top-level `sandbox/` fixture tree and not `tools/sandbox/`, and the card does not say so.
**Fix:** State the `configsync` rule, either its narrow carve-out applies to its test files or those files are out of scope.
If it applies, add `configsync_test.go` to a sweep card.
State that the `sandbox/` skip is the repo-root `sandbox/` tree only, and that `tools/sandbox/` is scanned.

### [NIT] Card 20's scan names its second spec type only by description

**Location:** `_lyx/plan/20-spawn-role-constants.md`
**Issue:** The scan covers `shuttleengine.Spec` and "reed's strand-spawn spec type, whatever struct `reedengine`/`reedcli` take a `Role` in".
That leaves the implementer to find the type.
The type is `reedengine.AddSpec`, whose `Role` field is at `internal/reedengine/strand.go`.
**Fix:** Name `reedengine.AddSpec` in the card.
