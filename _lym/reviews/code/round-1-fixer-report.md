---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed-differently
    reason: "The door is one-way as proposed. The blast radius is not `loom, shed and batten`: contracts/specs/loom-final-summary-spec.md:42 requires one word and the description validator refuses more, so it reads `runs`, with the two records and the three modules named in prose below."
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed-differently
    reason: "The proposed interview line says the log was missing when the record was seeded, but this path runs only over a record that already existed. The line reads: The support log was missing beside an existing record; the record was kept as it stood."
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed-differently
    reason: "An already-cancelled context returns from the entry check at internal/loomshed/recordwrite.go (entryErr) and never reaches the check before Stuck, so the proposed test would pass with that check removed. The test cancels from inside the ReadStored seam instead."
  - id: F12
    disposition: fixed-differently
    reason: "The proposed sentence reads as the scope of the whole change, which also adds the jacquard recipe and the Expand rule. It is limited to Record-Seed."
---

# Round 1 fixer report

## Changes

- **F1:** `loomshed.SeedParentParam` is exported and used by `recordSeedDeps`, `loomSeedFor` and Record-Seed's reason.
  From the self-check, also by `childSeedParams` in `internal/battencli/wire.go`, `disagreeingSeedRefusal` and the missing-seed remedy in `internal/loomcli/arm.go`.
- **F2:** `ReadSeedErrorIsReturned` asserts the step text and the way forward.
- **F3:** the shed recipe spec states the run-ending block rule, the seed-row clause of the loom contract and the composed recipes.
- **F4:** `TestRecordSeedDeps_ReadsTheRunsSeed` in `internal/loomcli/wiring_test.go`.
  It builds the location by hand, checks `SeedPath` by statting the written seed, and records driver `llm` so a missed read cannot match a default.
- **F5:** Merge danger in the change description (see the disposition).
- **F6:** the arras header names the jacquard header.
- **F7:** the `loomshed.go` and `doc.go` comments name the loom recipes.
  From the self-check, also one comment each in `internal/loomrecipe/sequence_test.go` and `internal/loomcli/smoke_test.go`.
- **F8:** the jacquard comment names `motif-recipe.yaml`'s row.
- **F9:** the stored-only mode writes a support log that claims no source; the three logs share one renderer.
  The `Call` and `NewRecordSeed` docs and the change description say so.
- **F10:** one admitted and one refused nested-block row in `TestExpand`.
- **F11:** `CancelDuringTheCallWinsOverStuck`.
- **F12:** the scope sentence no longer names the task record.

## Verification

- `lym gate test` (tier0) passed over `loomshed`, `shedbuild`, `loomcli`, `loomrecipe`, `shedrecipe` and `contracts/...` after the review fixes,
  and over `loomshed`, `loomcli`, `battencli` and `loomrecipe` after the self-check fixes.
- `lym loom lint-comments` and `lym loom validate-description` passed.
- `internal/loomcli/smoke_test.go` changed in a comment only; the tiers that build it were not run.
- No new test was run with its defect put back in.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW, `childSeedParams` still spells `parent`: fixed.
- fix-consistency, LOW, `disagreeingSeedRefusal` spells `parent`: fixed.
- fix-proposal, MEDIUM, F12's sentence reads as the whole change's scope: fixed.
- fix-proposal, MEDIUM, `childSeedParams` literal: fixed (same as above).
- fix-proposal, LOW, "it" in `doc.go` has no referent: fixed.
- fix-proposal, LOW, the kept-record log says the record was seeded: fixed.
- fix-affected-set, MEDIUM, `childSeedParams` literal: fixed (same as above).
- fix-affected-set, LOW, `disagreeingSeedRefusal` and the `arm.go` remedy spell `parent`: fixed.
  The cobra help examples in `internal/shedcli/seed.go` keep the literal as example text.
- fix-affected-set, LOW, two test comments name `tapestry-recipe.yaml` as holding the rows: fixed.

## Disputed

## Deferred

## Rulings
