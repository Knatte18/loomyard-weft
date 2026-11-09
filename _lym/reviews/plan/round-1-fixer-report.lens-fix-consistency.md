```yaml
lens: fix-consistency
outcome: reported
```

Check: the fix-consistency lens over the fixer's change set.

- **LOW** — card 4, the rewritten `docs/overview.md` bullet ("its adopt entry is rewritten whole: the hidden worker the call follows, the adopt record and its resume; the step order with the pause before the owner write…").
  Card 4 lands before card 6, which builds the step order and the pause, and before cards 7 and 8, which build the worker, the record and the follow.
  At card 4's landing the overview would describe behaviour the code does not have yet.
  The twin place for that rewrite is card 10, which already rewrites the adoptcli `doc.go` over cards 4 to 8 but does not list `docs/overview.md`.
  Evidence: card 10's `**Edit:**` lists `PATTERN.md`, `pattern/PATTERN-dobby-runs.md` and `internal/adoptcli/doc.go` only, and card 4's Intent opens "D8: a green adopt check only reports".
  Card 4 could drop only the green-end sentences, with card 10 rewriting the adopt entry.
  The review's F6 put the edit on card 4, but card 4 is not the card where the behaviour exists.

The rest of the change set agrees with its siblings:
- No card still names `TestJudgeStanding`, `TestRunsInOrch`, `proc.IsAlive`, the wiring-guard claim or `pause_lifted`.
- `collect` exists for `envelopeOf`'s order.
- Card 2's `start` / `h.spawns` reword matches the harness.
- The test world already models standing move records, so card 7's new standing-move row can be built.
- Card 7's "stopped at `deploy`" row agrees with card 6's "stopped at `from`".
- Card 8's pointer to the package doc agrees with card 10's new envelope bullet.

Tool calls:
- One grep/sed call. It read the plan cards for stale references, adoptcli for `collect`, the `fakeStarter` in `internal/orchcli/start_test.go:40-60` and adopt_test's move-record support. My context lacked all four: the other plan cards, `collect`'s existence, the fake starter's recorded calls and the test world's fields.
