```yaml
lens: fix-claims
outcome: reported
```

The two findings below come from two places where a sentence the fix added doesn't match the code at HEAD. I checked the other new claims and found them accurate.

- **MEDIUM. Card 3, instruction-source bullet:** The new sentence says the repository root "is the root `dotgit.FindRoot` returns for that resolved anchor, as `codexengine`'s `readCheckout` derives it". That isn't how `readCheckout` works.
  - `readCheckout` is called with the raw `c.dirs.Anchor`. See `codexengine.go:331` and `home.go:241`.
  - It runs `dotgit.FindRoot(anchor)` on the unresolved anchor at `home.go:74`, and only then resolves the root with `filepath.EvalSymlinks(root)` at `home.go:78`.
  - Finding the root on the resolved anchor is a different derivation, so the "as `readCheckout` derives it" claim is false. It also undercuts the card's goal that both engines key files the same way.
  - Fix: run `dotgit.FindRoot` on the anchor and resolve the root it returns with `filepath.EvalSymlinks`.
- **MEDIUM. Card 1, the `TestPrepare_AppendsStandingPassages` bullet:** The new sentence says the `outside_hub`, `no_reader` and `reader_error` rows each give the zero `Standing` "with the Warn log". Only one of those rows checks a log.
  - In `claudeengine/prepare_test.go:618-636`, only `reader_error` sets `wantLogged` (`"lym heddle list"`).
  - `no_reader` checks only the launch text, and `outside_hub` checks `wantNoCalls`; neither checks a log.
  - The card's own Intent says only a reader error is "still logged at Warn".
  - Fix: tie the Warn log to the `reader_error` row only.

The two other claims I verified against HEAD:
- Every `seatengine.Seat{}` return in `multiLLMSeat` sets no `Stencil`; only the final literal does, at `entries_multillm.go:164-166`.
- `seatengine.New` only stores the shuttle (`engine.go:70-71`), so building the engine with no shuttle is safe while `SeatTables` runs no seat.

Tool calls:
- Grepped `seatengine.New` and the `readCheckout` callers, because my context had `readCheckout`'s first line but not its callers or `seatengine.New`'s body.
- Read `claudeengine/prepare_test.go:586-665` to check what the three rows assert, because my context had only the row names from a grep.
