## Goal

```markdown
## Seen

A whole tier 1 pass starts 196 test binaries, and 130 of them hold no tier 1 test (`docs/research/speed/tier1-test-revision-2026-11-06/report.md`).
Go still compiles and links each before it finds nothing to run.
On neve, starting only the 66 with a tier 1 test takes the floor from 20.5 s to 12.4 s, about 8 s off every pass, with no test lost.

On OSL-1046 it should weigh far more, since Cortex XDR holds a binary it has not seen for about ten seconds.
At four binaries at a time that is 65 s a pass if a fifth of the 130 are held and about 325 s if all are; neither figure is measured.
Whether a rebuilt binary counts as new depends on whether Cortex keys on content or on path, and `go test` puts each binary in a new temporary directory.

## Idea

The gate derives, per tier, the packages that hold a test of that tier, and passes only those to `go test`.
It applies to every tier, not tier 1 alone (operator, 2026-11-07).
The list is derived, so a package gaining its first test of a tier enters without a hand edit.

## To settle

One gate pass on OSL-1046 with `lym gate report`'s launch times would show the holds.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-07): Opt-in {test_targets}; the doc example shows this hub's tiers after the switch

Decision: Keep {test_targets} opt-in in the code. The gatecli/doc.go example shows loomyard's gate.yaml tier0 to tier2 as they read once switched to {test_targets}; the landing names that hub step (lym config gate --set for tier0, tier1 and tier2), and the parent relays it to the operator.

Rationale: Tier commands are each hub's own; lym cannot rewrite a hub's go test command, and the hub's current tiers already match the example but for {test_targets}.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
