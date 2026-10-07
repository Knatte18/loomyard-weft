---
round: 3
focus:
  - "Target completeness: for each card that changes an `Env` field, a registered stencil, a seam signature or a package's top-level test count, check the card lists as targets the files that enumerate or count what it changes. Settle whether the plan names `cmd/lyx/testdata/test-budget.yaml` (checked by `TestTestBudget_PackagesStayWithinBudget` in `cmd/lyx/testbudget_test.go`) for every added top-level `Test` function, and a row for any package that gains its first test file."
  - "Env seam fixtures: card 8 adds `Env.BurlerRemover` and requires it in the BurlerRound row. Settle whether the plan fills it in `internal/testkit/envkit/envkit.go` (`FullEnv`), `internal/shedrecipe/fixture_test.go` (`newTestEnv`) and handles it in `internal/battencli/wire_seams_test.go` (`intentionallyNil`)."
  - "Stale docs: card 7 replaces the single round orchestrator with two orchestrators and more instruction files. Settle whether every doc under `docs/` that describes the burler round's orchestrator or instruction files is listed as a target, including `docs/shared-libs/stencil.md`."
  - "Start-failure reporting in card 7: `StartGated` in `internal/shuttleengine/run.go` returns `(nil, err)` on a failed start, unlike `RunGated`, which built a Result. Settle what the plan says a half that fails to start reports (outcome class, identity, run dir, error text) and whether a test pins it."
  - "New refusals reachable through a loom verb or `lyx burler run`: for each refusal a card introduces or widens, settle whether its message ends with a way forward, a `contracts/specs/refusal-spec.md` row lands in the same card, and a test reaches it (`PATTERN-refusal-way-forward`)."
  - "Real-adapter coverage: for each exported adapter a card gives a contract, such as `RunnerShuttle` and its nil-not-typed-nil returns, settle whether some hermetic test reaches the adapter's own code rather than only a fake."
---

Focus entries name where to look; the reviewer still applies the whole rubric.
