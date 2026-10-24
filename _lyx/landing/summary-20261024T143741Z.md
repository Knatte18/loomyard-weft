# gate test: a --card scope vets the card's packages under other tier tags

A `lyx gate test --card` call now also runs `go vet -tags tierN` over the card's packages, so a compile break under another tier's build tag fails at the card.
Before, a helper defined in a tier0 file and used by a tier1 or tier2 file passed every card gate and failed only in the batch gate's vet steps, which cost a separate fix pass across cards.

Closes #865

## What changes

Under the `go` scoper, a `--card` plan gains one vet run per qualifying tier, after the listed tiers and before the guard step.
A tier N qualifies when some `.go` file in a card package matches the build constraints under the single tag `tierN` and not under no tags, on the host's GOOS and GOARCH.
A tier is skipped when a listed tier is N or covers N, since that tier's own command already compiles the tag set.
No vet run is planned for the `none` scoper or for the computed, `--base`, `--all` and explicit-target forms.

Each vet run is a tier run of its own:
- It holds one gate slot and runs in the tier's module directories with the tier's `env`, `parallel` and `timeout_min`.
- It uses the go binary the gate builds `lyx` with and goes through the go-json counter.
- It writes a run record and a summary line with scope form `vet`, and takes part in the green record under its own fingerprint.
- A failing vet run stops no later run, and the closing line names it `vet tierN`.

Because the step reuses the existing slot, timeout, record and carry paths, PATTERN-gate-slot and PATTERN-gate-carry hold without a new path.

## Design decisions

- The step is built into the verb, with no new `gate.yaml` key, so every hub's `card` key gains it without a config edit.
  The `go` scoper already reads `tierN` tags for the guard scan and the empty-tier check.
- The tag scan runs in `computeScope`'s card branch and is carried on the scope as `tagTiers`, so `planTierRuns` stays disk-free.
- A tier that adds no file under its tag is not vetted, so a package with no tagged file costs nothing.
- Bound: a break visible only under a combination of tags, or under a tier that adds no file, is not caught at the card and stays the batch gate's to find.
- A hub whose tier N does not compile under `tierN`, such as a non-Go tier numbered like a Go one, gets a vet run it did not ask for; the `go` scoper already assumes the tag naming.
- The batch key, also of scope card, gains the step for tier2 and tier3 tagged files, which tier1's command in this hub already vets, at the cost of a second, mostly cached vet.

The change is in `internal/gatecli` (`scope.go`, `tiers.go`, `run.go`), with `internal/gatecli/doc.go` and `docs/overview.md` updated to describe the step.

## Tests

- `tiers_test.go` covers the plan: vet runs for tagged tiers, none for listed or covered tiers.
- `scopes_integration_test.go` covers a `--card` call over a package whose tier1 file uses a tier0 helper (non-zero exit, `vet` scope form, `vet tier1` in the closing line) and the same call over a package with no break (passes).

Not run by hand: a `--card` gate against a real hub's card with retagged files.
