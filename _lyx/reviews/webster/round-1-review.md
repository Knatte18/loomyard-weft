---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: internal/loomengine/config.go:551
    summary: The new "models.yaml unreadable for discussion_advisors" refusal has a spec row but no test reaches it, which PATTERN-refusal-way-forward requires
  - id: F2
    severity: LOW
    class: consistency
    location: internal/shedrecipe/recipe.go:140
    summary: Doc comments naming the DiscussionWrite entry as the only reader or caller of a shared seam are false now that the DiscussionSeats entry reads the same seams
---

Range reviewed: `git diff $(git merge-base main HEAD)..HEAD` (no rework round exists; `product.parent` is `main`), nine commits for cards 1 to 9.
`lyx gate test ./...` passes on the head, and the changed packages pass on their own.
The seams the focus directive names hold: `DiscussionSeats` is out of the coverage-guard allowlist and in `loomrecipe.RecipeEngines`, the engine-set want set and the gate-quiescence writer cases; `applyDiscussionProducer` changes only the engine of the row named `loomshed.NameDiscussionWrite`; `wire` fills `DiscussionTable` in card 7 and `DiscussionSeats` in card 8.
`LoadConfig` normalises the four empty spellings, names key and 1-based index in its refusals, and loads the registry only for a non-empty list.
The chair stencil includes the discussion stencil whole, and the discussion stencil's body changes only in its stripped header comment.
`stampGate` evaluates the gate once and leaves a result that carries its own `GateOutcome` alone.
No Edit group re-signs a member, so no `caller-uncovered` check is in play; Create and Prosa groups carry no mechanical check.
The budget rows rise by exactly the new top-level tests per package, and `cmd/lyx` is green.

### [BLOCKING] An untested config refusal breaks PATTERN-refusal-way-forward

**Location:** `internal/loomengine/config.go:551`, `internal/loomengine/config_test.go`
**Issue:** `LoadConfig` returns a new refusal when `modelspec.LoadRegistry` fails for a non-empty `discussion_advisors`, with the way forward "fix models.yaml, or set the key to an empty list for no advisors", and card 3 adds its row to `contracts/specs/refusal-spec.md`.
PATTERN-refusal-way-forward requires that a new refusal in loom lands with its row and that a test reaches it.
The other new refusals are reached by `TestLoadConfig_Refuses` rows (entry shape, unknown alias, mapping value, unknown `discussion_producer`), but nothing in `internal/loomengine/*_test.go` writes an unreadable or malformed `models.yaml`, so this refusal and its row are unreached.
**Fix:** add a `TestLoadConfig_Refuses` row, or a subtest beside it, that writes a malformed `models.yaml` under the base directory beside a non-empty `discussion_advisors`, and asserts the error names `discussion_advisors` and the "fix models.yaml" way forward.
A second assertion in the same test that an empty advisor list loads with the same malformed file would pin the "registry never read for an empty list" bound.

### [LOW] Doc comments name the DiscussionWrite entry as the only user of seams the DiscussionSeats entry now uses

**Location:** `internal/shedrecipe/recipe.go:140`, `internal/shedrecipe/recipe.go:155`, `internal/shedrecipe/entries_gate.go:1`
**Issue:** `Env.ParentReview` says "Only the "parent-review" gate and the DiscussionWrite entry read it", but `discussionSeatsEntry` reads `env.ParentReview.Store` for its preparation.
`Env.CommitDiscussion` says it is invoked by "the DiscussionWrite entry's commit decorator", but `discussionSeatsEntry` wraps the same closure in the same decorator.
The file header of `entries_gate.go` lists the gate-capable entries that call `resolveGateSpec` and omits `DiscussionSeats`, which calls it.
Card 7 edited `Env` and the entry files and left these three statements false.
**Fix:** reword each to name the group instead of enumerating current members (for example "the entries that wrap a discussion producer"), so the next entry does not make them false again.
