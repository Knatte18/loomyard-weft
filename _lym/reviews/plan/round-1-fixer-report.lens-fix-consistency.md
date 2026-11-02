```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of this round's change set (cards 1, 3, 7, 9 and the overview).

1. LOW — card 9, Tests: `TestSpecFor_PreGotoEndsFixers` was dropped and nothing replaced it, so no test row pins the `specFor` wiring any more.
   - The lens-review record says that test was added to close the coverage finding "card 9's `specFor` wiring and no-location receiver have no row" (`_lym/plan/lens-review.md:40`, from `lens-review.lens-coverage.md:70-72`).
   - In the new card, `TestFixerEndingGuard` tests the wrapper alone. Nothing asserts that `specFor` (location branch), `goDriverDepsFor`'s `ConsumeTackler` request or `loomConsumeAttempt` actually set `fixerEndingGuard`. Nothing pins that a no-location receiver leaves `GotoGuard` unwrapped or nil either.
   - So the change undoes an earlier fix and leaves the plan's own record of it stale.
2. LOW — card 9, the `endGateFixers` bullet against the `fixerEndingGuard` bullet: the way forward no longer fits the consume path.
   - `endGateFixers` still gives the way forward "transient, re-run the goto" and "re-run the goto once the named process is gone".
   - Its error now reaches the user only through `checkGuard`, which wraps it as "goto target %q could not be checked: %w; way forward: re-run goto as …" (`internal/shedengine/goto.go:30`).
   - On the consume path, this round's change routes the error through the Relaunch guard. Before, it was returned as the consume's own error, whose way forward was "act on the refused stop as a new stop" (`internal/sheddriver/tacklerconsume.go:23`).
   - So a consume refusal now carries two goto way-forwards and no consume one, and the card does not say so.

No other sibling disagreement:
- The overview line 24 edit matches card 9.
- D8 ("before it moves the run") still agrees with "before it writes the move".
- The card 1, 3 and 7 additions match their findings, and no other card names `loomPreGoto`, `Hooks.PreGoto` or `endRunFixers`.

Tool calls:
- One grep over `_lym/plan/*.md` for references to `loomPreGoto`, `PreGoto`, the dropped test, `endRunFixers` and runproc doc. My context had no sibling cards or lens records beyond the four cards and the overview.
