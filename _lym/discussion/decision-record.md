## Goal

```markdown
Imported from [issue #273](https://github.com/Knatte18/loomyard/issues/273).

**What happened.**
A plan created a function in one card and gave it its first production caller in a card of a later batch.
The test-only-code guard (`cmd/lym/testonlycode_linux_test.go`) therefore failed the first card's gate.
The implementer added a temporary `testOnlyCodeAllowed` entry, with a `Loosened-Check` trailer, to pass.
The guard also fails on a stale entry, so the later card that wires the caller has to remove it,
but that card's targets did not list the guard file.

**Why it is abnormal.**
The plan as written could not pass its own gates.
The implementer had to loosen a check, and the removal fell outside every card's declared targets.

**What to do.**
Have plan writing or plan validation handle a symbol whose first production caller is in a later card:
put the declaration and its first caller in one card, or list the guard file in the targets of both cards,
or let the card gate defer this guard until the card that adds the caller.

Provenance: task `dobby-run-watch-build`, note `01-run-watch-setting.md`.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - group-filter-and-handle-ref-branches-untested (class design, severity MEDIUM, open in rounds 1)
  - test-helper-and-test-support-funcs-refused (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
