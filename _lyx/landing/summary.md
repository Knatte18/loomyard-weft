# gate test: a --card scope vets its packages under other tier tags

A helper defined in a tier0 file and used by a tier1 or tier2 file compiled under the card gate, because the card gate never built the card's packages under the other tiers' tags.
The break surfaced only in the batch gate's vet steps, far from the card that caused it.

`lyx gate test` now plans a vet step for a `--card` scope under the `go` scoper.
The step runs after the listed tiers and before the guard step, and names a failure `vet tierN` in the closing line.

Closes #865

## Design

- The step is built into the verb with no new `gate.yaml` key.
  The `go` scoper already reads `tierN` build tags, so every hub's `card` key gains the step without a board edit.
- Tier N is vetted only when some `.go` file in a card package matches the build constraints under the single tag `tierN` and not under no tags, and no listed tier is N or covers N.
  That is the case where a tag-set compile differs from the listed tiers' compiles; a package with no tagged file costs nothing.
  The scan runs in `computeScope`'s card branch and rides on the scope as `tagTiers`, so `planTierRuns` stays disk-free.
- Each vet run is one run per tier: `go vet -tags tierN` over the card's packages in each module directory.
  It holds one gate slot, uses the tier's `env`, `parallel` and `timeout_min`, and goes through the go-json counter.
  It writes a run record and summary line with scope form `vet`, takes part in the green record under its own fingerprint, and a failure stops no later run.
  It reuses the existing slot, timeout, record and carry paths, so PATTERN-gate-slot and PATTERN-gate-carry hold without a new path.
- Bound: a break visible only under a combination of tags, or under a tier whose tag adds no file, is not vetted at the card and stays the batch gate's to find.
- Only the `--card` form is changed.
  The computed, `--base`, `--all` and explicit-target forms and the `none` scoper plan no vet step, since their keys list tier1, whose command already vets each tag set in this hub.
- The batch key, also of scope card, gains the step for tier2 and tier3 tagged files, at the cost of a second, mostly cached vet.
- A hub whose tier N does not compile under `tierN`, such as a non-Go tier numbered like a Go one, gets a vet run it did not ask for; the `go` scoper already assumes the tag naming.

The change lives in `internal/gatecli` (`scope.go`, `tiers.go`, `run.go`), with `internal/gatecli/doc.go` and `docs/overview.md` updated to describe the step.

## Tests

Added: planning tests in `tiers_test.go` and integration tests in `scopes_integration_test.go` covering the plan order, the skipped cases, and a `--card` call over a package whose tier1 file uses a tier0 helper, which fails with `vet tier1` while a clean package passes.

Not run by hand: a `--card` gate in a real hub against a retagged card, to see the vet step's timing and cache behavior.
