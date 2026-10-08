---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/03-batten-claim.md (Test paragraph)
    summary: Card 3 copies the hub-with-board-config fixture from loomcli's landingdeps_markdone_test.go into battencli, a second package-local copy of shared test support that PATTERN-testkit requires to live in one kit
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/03-batten-claim.md (Test paragraph, done and absent rows)
    summary: Card 3's done-entry and absent-slug rows re-pin the skip that Card 2's TestSetRunStatus already pins, since the skip lives in SetRunStatus and batten only calls it
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/01-run-lock.md (Tests paragraph, passing rows)
    summary: Card 1's every-write rows for entries with no, empty, done, abandoned or active status re-pin existing write behavior that TestUpsertTask, TestPromoteMakesNoteATaskAndIsIdempotent and TestPruneRemovesDoneEntryAndDesignDocKeepsAbandoned already cover
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/03-batten-claim.md (Test paragraph)
    summary: the acceptance criterion that the claim survives a failing Topology.Add has no test, since Card 3's test fails the row at the fabric config load before Add is reached
  - id: F5
    severity: NIT
    class: consistency
    location: _lyx/plan/01-run-lock.md (Intent, boardCriticalSection bullet)
    summary: the parenthetical listing every mutating Board method omits SetStatus and RetireLegacy, which also go through boardCriticalSection, so the enumeration is already wrong
---

### [BLOCKING] Card 3 duplicates a shared test fixture across packages

**Location:** `_lyx/plan/03-batten-claim.md`, the Test paragraph: "a wired prime whose temp hub holds only the board config (written as `landingdeps_markdone_test.go` does)".

**Issue:** `markDoneFixtureDir` in `internal/loomcli/landingdeps_markdone_test.go` is the one place today that writes `boardengine.ConfigTemplate()` into `configengine.ConfigFile(fabricengine.BoardDir(hub), "board")` to build a hub that holds only the board config (grep for `boardengine.ConfigTemplate()` across `_test.go` files finds only it).
Card 3 tells the implementer to write the same fixture in `internal/battencli/wire_seams_test.go`.
`PATTERN-testkit` says test support used by two or more packages lives in one kit under `internal/testkit/<kit>/` and is never duplicated per package.
Constraint violations are BLOCKING, as the round's constraints section requires.

**Fix:** have a card before or at Card 3 create a kit fixture, for example `plan:internal/testkit/boardkit#HubWithBoardConfig` -> `func HubWithBoardConfig(t *testing.T) string`, which writes the board config template under a temp hub's board directory and returns the hub path.
Add `internal/loomcli/landingdeps_markdone_test.go` to that card's `Edit:` so `markDoneFixtureDir` uses it, and have Card 3 list it under `Uses:` and build its test on it.
The kit imports only `boardengine`, `configengine` and `fabricengine`, with no `internal/*cli` and no `os/exec`, so it meets the kit import rules.

### [LOW] Card 3's done and absent rows repeat Card 2's TestSetRunStatus

**Location:** `_lyx/plan/03-batten-claim.md`, Test paragraph rows "a `done` entry stays `done`; an absent slug creates no entry".

**Issue:** Card 3's closure calls `SetRunStatus`, and Card 2 moves the absent-or-done skip into it.
Card 2's `TestSetRunStatus` already has rows for "a `done` entry (left `done`, no write)" and "an absent slug (nil error, no entry created)".
The two batten rows re-pin that same behavior one layer up.
Batten's own new behavior is that the closure writes the claim through `SetRunStatus` before anything else, and that a failed write only warns.
The no-status row and the no-board-config row already cover that behavior.

**Fix:** drop the `done` and absent rows from `TestWire_CreateWorktreeClaimsTheBoardEntry`, and keep the no-status row and the no-board-config row.

### [LOW] Card 1's every-write rows on unlocked entries repeat existing tests

**Location:** `_lyx/plan/01-run-lock.md`, Tests paragraph: "and every write on an entry with no, empty, `done`, `abandoned` or `active` status".

**Issue:** that phrase asks for every write row against each of five non-run statuses.
Writes on entries without a run status are today's behavior, already pinned by `TestUpsertTask`, `TestPromoteMakesNoteATaskAndIsIdempotent`, `TestPruneRemovesDoneEntryAndDesignDocKeepsAbandoned` and the merge and set-deps tests.
The new behavior is only that these statuses do not lock.
The same test's `IsRunStatus` rows pin that, together with one row showing a scope-changing write passes on an unlocked entry.

**Fix:** replace "every write on an entry with …" with `IsRunStatus` rows for nil, empty, `done`, `abandoned` and `active`, and one scope-changing upsert row on an entry with a non-run status, which shows the check consults the predicate.

### [LOW] No test pins the claim surviving a failing Topology.Add

**Location:** `_lyx/plan/03-batten-claim.md`, Test paragraph.

**Issue:** the decision record's acceptance criteria include "When `Topology.Add` fails after the claim, the entry keeps `running · Worktree-Create`".
Card 3's test fails the row at `fabricengine.LoadConfig`, before the already-complete probe and `Add`.
It proves the claim survives a failure that comes before `Add`, but no row reaches an `Add` refusal.
`internal/battencli/lifecycle_integration_test.go` already drives `c.env.CreateWorktree` to an `Add` refusal (`stepCreateRow_LeftoverBranchIsRewordedForPrime`) over a hub that carries a board config.

**Fix:** add `internal/battencli/lifecycle_integration_test.go` to Card 3's `Edit:`.
In the leftover-branch step, seed the slug's board entry before the call and assert it reads `running · Worktree-Create` after the refusal.
This extends an existing step, so it adds no top-level test.

### [NIT] Card 1's list of mutating Board methods is incomplete

**Location:** `_lyx/plan/01-run-lock.md`, Intent, `boardCriticalSection` bullet: "Every mutating Board method (`UpsertTask`, `UpsertTasksBatch`, `RemoveTask`, `MergeTasks`, `SetDeps`, `Promote`, `Prune`, `ImportIssue`) goes through it".

**Issue:** `SetStatus` and `RetireLegacy` also run through `boardCriticalSection` (`internal/boardengine/board.go:114`, `:315`), and so does `Rerender`.
The list is a perishable enumeration that is already incomplete.
The claim it supports, that every caller gets the lock, does not depend on the members.

**Fix:** drop the parenthetical, and keep "Every mutating Board method goes through it".

## Verdicts on the focus directives

- Card 1 targets: `internal/boardengine/doc.go`, `internal/boardcli/doc.go` and `plugins/ly/skills/board/SKILL.md` are docs whose edits name no symbol.
  `internal/boardcli/intake_test.go` is a test file.
  The card also carries the member glyph `Board.boardCriticalSection`, so the member-glyph rule does not fire.
  `boardCriticalSection` keeps its signature, so it takes no re-sign arrow.
  Not a defect.
- Card 1 sizing: the docs and the skill text must land in the same commit as the lock (CLAUDE.md docs rule, decision record Constraints).
  The intake row pins Decision 6 through that same lock.
  One reviewable unit, not a defect.
- Test economy: the `intake_test.go` row is the only test of Decision 6's no-GitHub-write disposition, and Card 1's engine rows never drive `ImportIssue`, so it is not redundant.
  Card 2's locked-entry row pins that `SetRunStatus` replaces a non-`done` run status, which is behavior of that method, not of the lock.
  F2 and F3 cover the repeats.
- Card order: Card 2's locked-entry row relies on Card 1's behavior, and Card 1 comes earlier.
  Card 3 uses symbols from Cards 1 and 2.
  `openHubBoard`'s only callers are `internal/loomcli/wiring.go:182` and `internal/loomcli/landingdeps.go:104`, both inside Card 2's edited members.
  Not a defect.
- Attack surface: Card 1 bounds the unguarded `set-status` and status-only passes with the refusal's `lyx batten status` check.
  Card 3 bounds the warn-and-proceed path: on a failed claim, the lock window stays open until loom's first transition, as today.
  Both bounds are stated, so not a defect.
- Fidelity: every Decision and Constraint is carried.
  Decision 5's two ways forward and the no-run outcomes are in `RunLockedError.Error()`.
  The batch listing one slug twice and the merge with `set_status` are judged on the combined net change, and both are test rows.
  The intake fold refuses before any GitHub write.
  Decision 10 is in Shared Decisions.
  `OpenHub` is licensed by Decision 9.
  `SetRunStatus` moves loom's existing skip under the lock and only writes `status`, which Decisions 3 and 8 license.
  No unlicensed scope.
- Verify: `./...` covers `boardengine`, `boardcli`, `loomcli` and `battencli`, `integration` runs, `tmux` and `llm` are vetted, and the order matches the template.
  Not a defect.
- ImpactSummary: Card 1 states a real conclusion: one in-memory check per write, and only scope-changing writes to run-held entries change outcome.
  Card 2's names both of `openHubBoard`'s callers through their writes and concludes no observable change.
  Not a defect.
