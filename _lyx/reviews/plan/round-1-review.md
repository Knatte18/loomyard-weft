---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/04-review-budget-from-config.md, _lyx/plan/09-checkpoint-judging.md
    summary: The shared envkit.FullEnv builder is in no card's targets, so the new required Env fields ReviewMaxBounces and ReviewCirclingCheckpoint are set only in per-package fixtures
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/plan/03-loom-config-keys.md
    summary: docs/overview.md enumerates loom.yaml's keys and the budget text, but no card that adds the keys or moves the budget edits those lines
---

### [MEDIUM] envkit.FullEnv is not targeted by the cards that add required Env fields

**Location:** `_lyx/plan/04-review-budget-from-config.md`, `_lyx/plan/09-checkpoint-judging.md`

**Issue:**
Card 4 makes `loomrecipe.New` and `Routing` refuse `ReviewMaxBounces` below 1, and Card 9 makes the Bouncer entry require a positive `ReviewCirclingCheckpoint`.
Both cards say "test fixtures that build an `Env` set it" and name per-package files: Card 4 names `loomrecipe/fixture_test.go`, `shape_test.go`, `recipe_test.go` and `resume_test.go`, and Card 9 names only `shedrecipe/fixture_test.go`.
`internal/testkit/envkit/envkit.go` is the shared `Env` builder (`FullEnv`), and the loomrecipe fixtures and `shape_test.go` start from it (`envkit.FullEnv(t)`).
Its package doc says a new required field is added to the kit once, instead of to every package's private copy, and `PATTERN-testkit` forbids duplicating the kit's job.
Neither field is set by `FullEnv`, so every Env built from it fails the new positivity checks, and Card 9 leaves the loomrecipe fixtures without the checkpoint at all.

**Fix:**
Add `internal/testkit/envkit/envkit.go` to Card 4's `Edit` list (`ReviewMaxBounces`) and to Card 9's (`ReviewCirclingCheckpoint`), and state in each Intent that the kit sets the field once and the per-package fixtures set it only where a test needs a different value.

### [LOW] docs/overview.md lines on loom.yaml keys and the budget are not edited

**Location:** `_lyx/plan/03-loom-config-keys.md`

**Issue:**
The decision record's scope lists `docs/overview.md` (the circling and budget paragraphs), and CLAUDE.md requires docs in the same commit as the behaviour change.
`docs/overview.md` has a loom config line that enumerates the `loom.yaml` keys (`discussion_timeout_min` ... `parent_review_wait_min`).
Card 3 adds two keys and does not target `docs/overview.md`, so the enumeration goes stale; Cards 10, 13 and 15 edit the file but only for their own paragraphs.

**Fix:**
Add `docs/overview.md` to Card 3's `Edit` list and say in its Intent that the loom config line names the two new keys.

## Judged and not a defect

- Fidelity: D1–D12 and every Constraint are carried (D1/D2 by Cards 1, 2, 9; D3 by 3, 4; D4 by 10, 11; D5 by 5, 6, 7, 10; D6, D10 by 14; D7 by 8, 10; D8 by 12, 13; D9 by 6; D11 by 15; D12 by 16).
  No card scope lacks a licence.
- Attack surface: Cards 8, 9, 10 and 13 state what `accept`, repeated `continue`, the guard and `decision add` let through and what bounds each.
- Verify: `go test -tags integration ./...` covers every targeted package, and the smoke tier is compiled by `go vet -tags smoke`, which the rubric exempts.
- Granularity and sequencing: Cards 4 and 9 are large but each is one coherent change; config keys (3) precede their readers (4, 9), the notice field (5) and stencils (6) and record (7) precede Card 10.
