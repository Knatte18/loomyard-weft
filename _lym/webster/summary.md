# Loom names a contract, not a recipe: webster run summary

All 11 batches reached `done`, covering plan cards 1 through 20.
No batch needed recovery, a re-fork or a plan fix, and no digest carried a rename or a warning.
Deviations are the scope guard's list of files changed outside a batch's own targets; they are informational.

## Batches

- **01-discussion-fence-rule** (cards 1–4, HEAD `8d8e5856`): the discussion parser takes the top-level fence rule, a beam seed reads as an unknown recipe, Shed checks `contract: loom` at build, and retired config keys carry a way-forward text.
  Deviations: `internal/discussionparser/append.go`, `internal/shedbuild/loomcontract.go`, `internal/shedbuild/parse.go`.
- **05-shed-keys-in-shed-only** (card 5, HEAD `ae526444`): the arbiter, tackler and driver-kind keys leave `loom.yaml` and `darn.yaml` and are refused there.
  Deviations: `internal/configreg/runsettings.go`, `internal/darnengine/config.go`, `internal/loomengine/config.go`, `internal/loomengine/config_test.go`.
- **06-tapestry-config-module** (card 6, HEAD `90483ac0`): the plan and implementer keys move to a new `tapestry.yaml` module.
  The fork followed the card over the overview's shared decision, so `LoadConfig` never reads `tapestry.yaml` and `Config.WithTapestry` applies it.
  It also made `reviewbudget` always check the run-wide soft/hard pair, which the card did not ask for.
  Deviations: 16 files, mostly tests and doc comments in `configcli`, `configreg`, `configsync`, `hubreconcile`, `loomcli`, `loomengine`, `reviewbudget` and `shedrecipe`.
- **07-recipe-key-accessors** (cards 7–8, HEAD `3909ddf9`): `loomrecipe` answers a recipe's contract, keys, record engine, rows and retired rows by name, and one recipe-keyed `New`, `Routing` and `RetiredRows` replace the tapestry and darn pairs.
  The digest listed no deviations, though the fork reported test and doc edits outside the card targets.
- **09-drop-tapestry-preflight** (card 9, HEAD `126f5c14`): Tapestry-Preflight and its seed check go, and Preflight loads `batcher.yaml` for an Implementer recipe.
  Deviations: 12 files, mostly stale tests and doc comments, plus `internal/shedengine`, `internal/loomcli/bootstrap.go` and `contracts/stencils/orch/orch-template-role.md`.
- **10-drift-verify-before-review** (cards 10–11, HEAD `0f086296`): tapestry runs Drift-Verify before the Code-Review perch, and a Go Record-Write row writes darn's record and support log from the board entry.
  Deviations: 10 files, including the `internal/fsx` import allowlist entry in `internal/loomshed/seam_enforcement_test.go`.
- **12-darn-review-answer-key** (card 12, HEAD `fcdbe345`): darn's Code-Review perch measures against the decision record alone, with no snapshot.
  No deviations.
- **13-one-discussion-validator** (cards 13–14, HEAD `58f9a1ef`): one validator and gate serve every loom record plus Darn-Write's post-session guard, and Discussion-Write and Darn-Write share one run-and-commit wrapper.
  Deviations: 8 files in `loomcli`, `loomshed` and `shedrecipe`.
  The digest carried no departure, but the fork reported a `Pattern-Departure: PATTERN-fabric-vocabulary` trailer in `b3d5367f`.
- **15-darn-write-scope-guards** (card 15, HEAD `4b373e0e`): Darn-Write refuses a missing record, halts on its writer's stop marker, and stops writing the record.
  Departure from the digest: `PATTERN-fabric-vocabulary`; the restore way forward says "the committed copy under _lym" where the card says "the _lym weft repository's HEAD", since a bare `weft` token outside the owner set is forbidden.
  Deviations: `internal/darnengine/spec_test.go`, `internal/loomshed/darnwrite.go`, `internal/loomshed/darnwrite_test.go`, `internal/shedrecipe/entries_darnwrite_test.go`, `internal/shedrun/paths.go`.
- **16-decision-add-by-rows** (cards 16–17, HEAD `b469425a`): `decision add` keys on the recipe's rows and names reachable ways forward on a Record-Write record, and one Describe table and stencil serve every loom recipe.
  Deviations: `internal/landingshed/describe_test.go`, `internal/loomcli/decision.go`, `internal/loomcli/decision_test.go`.
  The digest carried no departure, but the fork reported a `Pattern-Departure: PATTERN-fabric-vocabulary` trailer in `475881de`, the same `weft` wording conflict.
- **18-recipe-keyed-wiring** (cards 18–20, HEAD `757f1200`): `loomcli`'s recipe-name branches read the recipe's rows and keys, `lym loom` refuses a non-loom run and a plan verb refuses a recipe without a plan, and `lym loom peers` reads each peer against its own recipe.
  Deviations: `internal/loomcli/arbiterprompt.go`, `arm.go`, `peers.go`, `peers_test.go`, `review.go`, `wiring_test.go`.

## Scope guard and open items

The scope guard flagged only the files listed per batch; the digests for batches 07 and 12 listed none.
Forks reported stale items the cards did not cover: `tools/tapestryrename/tables.go` still names `TapestryRetiredRows` and `NameTapestryPreflight`, `internal/hubreconcile/ensure_integration_test.go:556` mentions shed's `driver_kind`, and a comment at `internal/shedrecipe/entries_gate.go:124` says a darn run has no `PatternRecordPath`.
Card 13, 15 and 16 text said "weft repository" in way-forward messages, which conflicts with `PATTERN-fabric-vocabulary`; the plan wording needs the same fix.

## Verify gate findings

The gate failed on attempt 1 of 2 with a failing package, `internal/loomrecipe`.
Its findings file pointed at cards 06, 07, 09, 10 and 12 as touching that package.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `cbec60f4ca673423cb30d6acaf2430adaeec2eb1` fix: TestDarnVerifyBudget expects Darn-Write gates without the removed discussion gate
