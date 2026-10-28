```yaml
lens: fix-proposal
outcome: reported
```

fix-proposal lens: I challenged each reviewer proposal the fixer carried out as proposed (F1, F2, F3, F5).

1. **MEDIUM — card 18, idle-wake scenario.** Changed text: "the bell line is never typed: `events.jsonl` holds no `UserPromptSubmit` payload whose prompt equals `creelengine.BellLine`".
   Summary: the equality match passes vacuously on the defect this scenario exists to catch.
   Evidence: this session starts with a draft in its input box.
   If a defective guard types the bell anyway, `typeBell`'s guarded `Send` (card 11:38) appends it to the draft.
   The submitted prompt is then `<draft>[creel] Mail is waiting…`, which is not equal to `BellLine`, so the assertion still holds.
   The fix is to match payloads whose prompt contains `BellLine`.
   The interrupt-and-bell count should use the same contains rule so the two scenarios state their bell check the same way.

The other proposals hold:
- **F1:** advancing the fake clock past UTC midnight makes `append` call `rotate` (journal.go:223–226). `rotate` closes the file and then fails at `os.OpenFile` in the removed directory. It returns before `writeLine`, so nothing panics. The concurrent sweep timers in sweep.go:62–79 may also fire on `Advance`. Their appends fail the same way and leave `w.day` stale, so the row's `interrupted` append still fails.
- **F3:** only `Store.Interrupt` (verbs.go:529) and `inject --interrupt` (verbs.go:426) call `interruptTarget`, and no test matches these refusals with `errors.Is` or `errors.As`, so a typed error with unchanged messages breaks no caller.
- **F5:** the `shuttleengines` registry holds only `claude` (shuttleengines.go:49), and shuttle records `Provider` as the engine name (run.go:600). So a non-empty unregistered provider arises only in tests, and the new sentence holds.

Tool calls:
- grep of `clock.After`/`Now` in creelengine, the shuttleengines registry and `errors.Is`/`As` on refusals: to test F1's side effects and F3's callers, which my context lacked.
- listing of shuttleengines with `ToolBoundaryRenderer` and the run.go `Provider` field: to test F5's "only in tests" claim.
- read of the registry map in shuttleengines.go: to confirm which engines are registered.
