```yaml
lens: fix-proposal
outcome: reported
```

Task: fix-proposal lens, challenging the proposals the fixer applied as proposed (F2 to F7).

- **LOW**: `internal/lymcwd/gate_test.go`, the `TestCheckCwdAnchorGate` doc comment (F7). The added sentence, "A row with wantMessage also pins the refusal's whole text, cause and way forward included.", repeats the comment already above the table: "// wantMessage, when set, is the refusal's whole text." The same fact is now stated twice in one test, against the "say it once" rule. The F7 proposal asked for the sentence without accounting for the existing comment, so one of the two should go.

I found no breaking case for the other proposals:
- **F2:** the quotes keep a path with spaces runnable.
- **F3:** the new row expects the `backend` path as the `cd` target.
- **F4:** "may be refused" matches the gate's actual reach.
- **F5:** `levelOnly` still pins slug, order, target and heading level.
- **F6:** `agents` meets the spec's one-word rule.

Tool calls:
1. Searched `loom-final-summary-spec.md` for the blast radius rule, to check F6; the spec was not in my context.
