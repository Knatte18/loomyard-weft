## Goal

```markdown
Imported from [issue #274](https://github.com/Knatte18/loomyard/issues/274).

**What happened.**
A card added text to the `lym board upsert` help and pushed it past the 2000-byte cap that `TestCLIForm_EveryCommand` in `cmd/lym` holds.
Neither that batch's card gates nor its batch gate ran the test.
It first failed at a later batch's `--main-packages` gate, and was fixed there.

**Why it is abnormal.**
A change passed every gate of its own batch while breaking a test.
The failure landed on a different implementer, whose cards had not caused it.

**What to do.**
Make a scoped gate run this test when command help changes, for example by marking it `//lym:guard`
(its own doc comment says its checks are pure and run no invocation),
or by having the go scoper include `cmd/lym` when a package that registers commands changes.

Provenance: task `dobby-run-watch-build`, note `04-run-watch-wiring.md`.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-27): Close as overtaken by a69580709, no code

Decision: Option (a): a69580709 already puts cmd/lym in a card's scope when a card's package is one cmd/lym imports, so TestCLIForm_EveryCommand runs at the card and batch gates. Make no code change; do not add a //lym:guard mark. The change description names a69580709 as the fix and the remaining gap the plan rule covers.

Rationale: The bug is fixed on main; a guard mark that fixes no bug only adds weight to the gates, which the operator wants lighter.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
