# gate test: a passing --card call names the tiers it did not run

A card gate runs only the tiers the `card` key lists, so a card whose change breaks a higher-tier test passes it, and the batch gate finds the breakage a card late.
A passing card gate was easy to read as fully green.

Under the `--card` scope form, which the `card` and `batch` gate keys use, a passing `lym gate test` call now closes with one stderr line:

```
gate test: tier1, tier2 did not run, so this pass is not green for them
```

It names every tier `gate.yaml` declares that no listed tier is or covers, manual tiers included.
It is printed only when at least one such tier exists.

Closes #896

## Design decisions

- No tier1 run is added per card: a tier1 run per card costs more than the late find, and the batch gate still catches the breakage.
  The card key's tier list stays as `gate.yaml` sets it.
- The line is output only: no exit code, run record or green-record entry changes.
- A vet run under a tier's tag compiles that tier's files without running them, and a guard run runs only guard tests, so neither counts as running the tier (`notRunTiers` in `internal/gatecli/tiers.go`).
- A tier covered by a listed tier through `covers` is not named.
- A failing call prints no such line, since its closing failed-runs line already says it is not green.
- The computed, `--base`, `--all` and explicit-target forms print no such line: their tier lists are the coverage the hub chose for that site, and a line on every narrowed call would be noise.

## Docs

`internal/gatecli/doc.go` states the line, its tiers and its bound, and the Tier 1 bullet of `contracts/specs/loom-plan-spec.md` states that the card gate runs only its listed tiers and says so on a pass.

## Tests

The `--card` subtests in `internal/gatecli/scopes_integration_test.go` pin the line for a plain card call, leave out a tier a listed tier covers, and pin its absence on a failing card call and on the non-card forms.
Tests were not run by hand outside the gate.
