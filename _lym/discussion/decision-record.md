# Decision record — card-gate-misses-tier1

## Goal

A passing card gate says which declared tiers it did not run, so a card that passes tier0 is never read as fully green.

## Scope

- `lym gate test` under the `--card` scope form, which the `card` and `batch` gate keys use, prints one stderr line on a passing call naming the declared tiers the call did not run.
- The card gate's tier list stays as gate.yaml sets it: no tier1 run is added per card.
- `internal/gatecli/doc.go` and the Tier 1 bullet of `contracts/specs/loom-plan-spec.md` state the new line.

Out of scope: running higher tiers per card, changing any gate key's tiers, and the computed, `--base`, `--all` and explicit-target forms.

## Decisions

### D1 — A passing `--card` call names the tiers it did not run on stderr

After the plan's runs end and none failed, a call whose scope form is card prints `gate test: passed <run tiers> only; <not-run tiers> did not run, so this pass is not green for them` on stderr.
The not-run tiers are every tier gate.yaml declares that the call neither listed nor dropped as covered by a listed tier; a manual tier counts, since it did not run.
A vet run under a tier's tag compiles that tier's files without running them, so it does not count as running the tier.
The line is printed only when at least one declared tier was not run.
Rationale: the board entry asks for the card gate's result to state the higher tiers were not run; stderr is where the call's other closing lines go, and the line is beside the last tier summary the implementer reads.
Bound: the line is informational and changes no exit code, record or green-record entry.

### D2 — Only the card scope form prints the line

The computed, `--base`, `--all` and explicit-target forms print no such line.
Rationale: the board entry names the card gate, and the card form is the one the per-card and per-batch gates use; a line on every narrowed call would be noise on the darn and fix gates, whose tier lists the hub sets to the tiers it wants.

### D3 — A failing or halted call prints no not-run line

A call that failed already closes with the failed-runs line, and a halted call names that later runs did not run.
Rationale: the line exists so a pass is not over-read; a failure is not read as green.

## Constraints

- `PATTERN-gate-slot`: the gate's verdict and records stay as they are; the line is output only.
- `PATTERN-test-speed`: the test pinning the line is a row or assertion in the existing `--card` scope subtests of `internal/gatecli/scopes_integration_test.go`, at its existing tier.
- Docs land in the same commit: `internal/gatecli/doc.go` describes the line.

## Auto-mode assumptions

- The line goes to stderr, not into a JSON envelope: a passing `lym gate test` prints no envelope today.
- Manual tiers are named among the not-run tiers.

## Open risks

- An implementer may still summarise a passing card gate as green; the line makes the gap visible but does not force the report to say it.

## Acceptance criteria

- D1: a passing `--card` call over a hub declaring tier0 and tier1 with the card key listing tier0 prints the not-run line naming tier1 on stderr, and exits 0.
- D1: a tier covered by a listed tier is not named as not run.
- D2: a passing call under another scope form prints no not-run line.
- D3: a failing `--card` call prints no not-run line.
- D1, D2, D3: `internal/gatecli/doc.go` states the line, its tiers and its bound.
