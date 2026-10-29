# Card gate states the declared tiers it did not run

The card gate, the hub's `card` key over `lym gate test --card`, runs only the tiers that key lists, tier0 by default.
A card that breaks a tier1 test therefore passes its own gate, and the batch gate finds the break later, away from the card that caused it.
A per-card tier1 run costs more than the late find, so the tier lists stay as they are; instead the gate's result now says what it did not cover.

## What changes

- A `lym gate test` call under the `--card` scope form, which the `card` and `batch` gate keys use, that ends with no failed run closes with one stderr line: `gate test: tier1, tier2 did not run, so this pass is not green for them`.
- The line names every tier gate.yaml declares that no listed tier is or covers, manual tiers included.
  A vet or guard run under a tier does not count as running it, since it compiles that tier's files or runs only its guard tests.
- No line is printed when every declared tier ran or was covered, when a run failed, when the call halted, or under any other scope form.
- The line is output only: exit codes, run records and the green record are unchanged.

## Design decisions

- Only the card scope form prints the line.
  The computed, `--base`, `--all` and explicit-target forms serve sites whose tier lists are the coverage the hub chose for them, and a line on each would be noise.
- The line goes to stderr beside the failed-runs closing line, since a passing call prints no JSON envelope.
- A failing call prints no not-run line: it is not read as green, and its failed-runs line already closes it.

## Docs and tests

- `internal/gatecli/doc.go` states the line, the tiers it names and its bound.
- `contracts/specs/loom-plan-spec.md`'s per-card gate bullet says a card that breaks a higher-tier test passes the card gate and that the passing call names the tiers it did not run.
- `internal/gatecli/scopes_integration_test.go` pins the line on a passing `--gate card` call, leaves a tier a listed tier covers out of it, and checks a computed-scope pass and a failing `--card` call print none.
