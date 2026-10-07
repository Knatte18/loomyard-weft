# Round 2 fixer report

Every finding in `round-2-review.md` is fixed; `lyx loom validate-plan` exits clean (`ok: true`, informational findings only) over the edited plan.
`approved:` in `00-overview.md` is untouched (`false`).
The round-1 fixes are all still in place, and round 1 deferred nothing.

## Changes

- **F1 (BLOCKING, design: target lists miss forced files)**
  - Test budget: `cmd/lyx/testdata/test-budget.yaml` is now an Edit target of cards 1, 2, 3, 4, 6, 7 and 8, each raising its package's row by the top-level functions that card adds: `internal/shuttleengine` +1 (card 1), a new `internal/burlermarker` row of 1 (card 2), `internal/burlerengine` +1 (card 3), `internal/burlercli` +1 (card 4), `internal/burlerengine` +1 (card 6), +3 (card 7) and +2 (card 8).
    Card 6 gained an `Edit` group and an `ImpactSummary` for it.
    Cards 3, 7 and 8 now say which tests are rows of existing functions, so only the counted functions are new; card 5 says its claims are rows of the existing claims table; `internal/shedadapters`' one new function fits its headroom, so that row is unchanged.
  - `Env.BurlerRemover` fixtures: card 8 now edits `internal/testkit/envkit/envkit.go`, `internal/shedrecipe/fixture_test.go` and `internal/battencli/wire_seams_test.go`; both fixtures fill the seam with one shared `shedfake.StrandRemover` stub, and batten's `intentionallyNil` gains a `BurlerRemover` row.
  - Stale doc: card 7 now edits `docs/shared-libs/stencil.md` and rewords its `burler` bullet for two orchestrators and four instruction files.
  - Method: `00-overview.md`'s Shared Decisions gains a sentence requiring a card that changes an `Env` field, a registered stencil, a seam signature or a package's test count to list the fixtures, guard tests, budget row and docs that enumerate it.
- **F2 (MEDIUM)** — card 7 states what a half that never started reports under `StartGated`: `OutcomeDied`, `NotStarted`, empty identity, and a new `StartError` field on the half struct carrying the start error's text; the producer's double-failure message names it, and a row in the round-failure-rules test pins it.
- **F3 (MEDIUM)** — card 7 creates `shuttle_test.go`, one table-driven function over `RunnerShuttle(shuttleengine.NewRunner(…))` on `shuttlefake` seams (a started run returns a handle naming its strand, a failed start returns an interface-nil `Handle`), and card 8 adds the `ProbeGated` no-candidate row (interface-nil `Handle`, `found` false).
- **F4 (MEDIUM)** — card 3 gives `ResolveReview`'s per-entry refusals the key, the entry index and a way-forward clause, has the existing `refusal-spec.md` row cover the load-time and the resolve-time refusal, and has the `review_test.go` rows assert the clause.
- **F5 (MEDIUM)** — card 7's `ErrHalfNotStopped` error and card 8's failed-removal return end with `way forward: run "lyx reed remove <guid>", then re-step the row`; card 7 edits `contracts/specs/refusal-spec.md` to widen the `producer re-run: superseded strand not removed` row's trigger to a BurlerRound half that stays live, and the removal test rows assert the clause.
- **F6 (LOW)** — card 2's `burlermarker.Path` error for a review path outside the root ends with its way forward, and its test row asserts both paths and the clause; card 7 states that a standalone `lyx burler run` now refuses such a profile, says so in the verb's `Long` help, and adds a `burlercli` test row.

## Deferred

None.
