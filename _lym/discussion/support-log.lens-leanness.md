```yaml
lens: leanness
outcome: reported
```

Leanness lens on the drift-rework decision record and support log: what to move or cut.

- **MEDIUM — D2, last bullet.** The sentence "`compare` is required under `command` and refused under `lines` neither way" is garbled and reads as two opposite rules at once.
  Evidence: the next clause says `compare` is ignored under `lines`, but the bullet never states plainly what happens when `command` mode has no `compare` set.
  Fix: replace it with two clear rules. Under `command`, a missing `compare` is refused, naming the way forward. Under `lines`, `compare` is ignored and `lym drift status` reports that.
- **LOW — Notes for the plan writer, third bullet.** "`shedengine.RenameProducers` and a recipe's retired rows are not needed" repeats D4's "no rename or retirement is needed". Cut it.
- **LOW — Notes for the plan writer, first bullet, and D14's last sentence.** Both say the deletions go last, once nothing reads the removed records. Keep it in one place. The Notes bullet is the planning hint, so D14 can drop its sentence.
- **LOW — D9's Rationale, second sentence.** "removes the dirty-tree and verify-gate steps" repeats D14's removal list. Cut it from D9.
- **LOW — Auto-mode assumptions.** Each line restates a decision that already carries the pick, and the support log's Question ledger records the same picks. The section is required, so keep it, but it could be shortened to a pointer to each D-id rather than restating the pick.
- **LOW — Constraints.** Several bullets only point back at a decision ("`PATTERN-config-reinterpretation` and `PATTERN-config-layers`: D15", "`PATTERN-shed-stop-chain`: every drift Stuck goes through the stop chain", which D4 already says). Those can be cut. The bullets that add a rule not stated in any decision (strict agent-written files versus tolerant records; the general fabric seam naming no drift concept) should stay.
- **LOW — Scope "In".** It largely lists the decisions in order. It is a required section and short, so no move is needed; noted for emphasis only.
- **LOW — Open risks, last bullet (in-flight run with acceptances).** This is a consequence of D7's "no `expected.yaml` means an empty expectation", and it is stated as a risk without a bound. It belongs either in D7 as the stated behaviour for older runs, or it should stay as a risk naming what the parent does (`lym drift amend`). It must not be dropped, because it carries a behaviour the plan needs.
- **Kept on purpose (completeness check):** D1's rationale citing the `cs` trial format, and D6's `Warp-SHA` evidence. Both are facts the design rests on and belong in the record.

Tool calls: none.
