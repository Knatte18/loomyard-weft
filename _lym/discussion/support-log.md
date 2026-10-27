# Support log — darn-drift-await-row

## Interview

A darn run holds no interview; the writer reads the board entry and decides alone.

## Rejected alternatives

- Keep `CountedCapture`'s `awaitRow` parameter and pass true everywhere: rejected, since the board entry removes it once no caller needs the other branch.
- Remove `Deps.RunCapture` with `WaitCapture`: rejected, since Drift-Baseline's breakaway fallback still runs the capture in-process.
- Add a migration for a darn run parked between the drift rows: rejected by the operator's 2026-10-27 decision on the board entry.

## Review rounds

_No rounds yet._

## Question ledger

- Does `lym drift defer` need a change for darn? Self-picked no: it admits a blocked Drift-Await whatever the recipe.
- Does `CheckMainCapture` need a change? Self-picked no: it admits a blocked Drift-Await already.
- Does `RunCapture` go with `WaitCapture`? Self-picked no: the breakaway fallback uses it.
