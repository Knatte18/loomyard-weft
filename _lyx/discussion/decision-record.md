# Decision record — card gate vets the card's packages under the other tier tag sets

## Goal

A `lyx gate test` call over a `--card` scope surfaces a compile break under another tier's build tag at the card, instead of only in the batch gate's `go vet` steps.

## Scope

- `internal/gatecli`: a `--card` scope under the `go` scoper plans a vet step after the listed tiers and before the guard step.
- The vet step is one run per tier, of `go vet -tags tierN` over the card's packages per module, for each declared tier whose tag brings a file into a card package that a build without tags leaves out, and that no listed tier runs or covers.
- `internal/gatecli/doc.go` and `docs/overview.md`, where the gate's steps are described.
- Out of scope: the hub's `gate.yaml` (a hub-wide setting on the board), the computed, `--base`, `--all` and explicit-target forms, and the `none` scoper.

## Decisions

### The vet step is built into the gate, under the go scoper and the card form only

Decision: `lyx gate test` plans the vet step itself for a `--card` scope under the `go` scoper, with no new `gate.yaml` key.
Rationale: the `go` scoper already reads `tierN` build tags (the guard scan, the empty-tier check), so the step needs no config, and every hub's `card` key gains it without a board edit.
The other scope forms run under keys that list tier1, whose command already vets each tag set in this hub.

### A tier is vetted only when its tag changes the card's file set

Decision: tier N is vetted when some `.go` file in a card package matches the build constraints under the single tag `tierN` and not under no tags, and no listed tier is N or covers N.
Rationale: that is exactly the case where a tag-set compile differs from the listed tiers' compiles, which is the issue's "at least when the card touches build tags"; a package with no tagged file costs nothing.
Bound: a break visible only under a combination of tags, or under a tier that adds no file, is not vetted at the card and stays the batch gate's to find.

### The vet run is a tier run of its own

Decision: each vet run holds one slot of the pool, runs in the tier's module directories with the tier's `env`, `parallel` and `timeout_min`, uses the go binary the gate builds `lyx` with, goes through the go-json counter, writes a run record and summary line with scope form `vet`, takes part in the green record under its own fingerprint, and a failure stops no later run and is named `vet tierN` in the closing line.
Rationale: it reuses the slot, timeout, record and carry paths every run already has, so PATTERN-gate-slot and PATTERN-gate-carry hold without a new path.

## Constraints

- PATTERN-gate-slot: the vet build runs only inside a held gate slot.
- PATTERN-gate-carry: a vet run skips only through the green record.
- PATTERN-test-tiers: no site, stencil or gate key changes; the step is internal to the verb.
- PATTERN-tier-tags and PATTERN-test-speed for the tests added.

## Auto-mode assumptions

- The card gate is the `card` gate key, or any call with `--card`; the batch key, also of scope card, gains the step for tier2 and tier3 tagged files, which tier1's command in this hub also vets, at the cost of a second, mostly cached vet.
- A listed tier's own command compiles its tag set, and a tier that covers N compiles N's.

## Open risks

- A hub whose tier N does not compile under `tierN`, such as a non-Go tier numbered like a Go one, gets a vet run it did not ask for; the `go` scoper already assumes the tag naming.

## Acceptance criteria

- A `--card` plan under the `go` scoper holds one vet run per tier whose tag brings a file into a card package, after the listed tiers and before the guard step, over the card's modules and packages.
- No vet run is planned for a listed tier, a tier a listed tier covers, a tier whose tag adds no file, the `none` scoper or a scope form other than `--card`.
- A `--card` call over a package whose tier1 file uses a helper from a tier0 file fails with a non-zero exit, the vet run's summary line has scope form `vet` and the closing line names `vet tier1`; the same call over a package with no such break passes.

## Notes for the plan writer

- `planTierRuns` stays disk-free: the tag scan runs in `computeScope`'s card branch and is carried on the scope.
