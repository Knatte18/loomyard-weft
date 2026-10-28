```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fixer's F1/F2/F3 change set through the fix-code-correctness lens.

No findings.

- **F1, `stepChases` and `releaseGoneHolder`:** both call `listStrands()` before `s.do`, and both pass `listing` through. The only two callers of `settleUnansweredLocked` are updated, and `lym gate test` was green.
- **F1, strands read failure:** `listStrands` wraps through `readStrands`, which uses the same error text the old inline call had. With a strand sender, the notice returns `listing.err`, `settleUnansweredLocked` logs it as a warning, and the record is not undone. A program sender never reads the listing, so a failed read cannot block it. A failed read no longer blocks chases or settles, because the error only surfaces when the notice needs it.
- **F1, side effect:** the strands are now read on every `stepChases` call, including ticks where no step is due. That is one extra `Strands()` call per running tick, outside the mutex, and it is not a defect.
- **F2, new test:** if a wiring file reverted to `NotifyParent`, the cut notifier text would not contain `orchcli.NotifyAsker(`, so the test would fail.
- **F2, cut point:** `strings.Cut` at `"\n\t})"` matches the one-tab close of each `SetNotifier` body. `loomcli/wiring.go` has a later `NotifyParent` call at line 587, but it falls after the cut and cannot satisfy or confuse the check. Each wiring file has a single `runner.SetNotifier(`.
- **F2, working directory:** the relative `../<pkg>/wiring.go` paths resolve because `go test` runs in the package directory.
- **F3:** a doc-comment edit only.

Further tool calls: none, the context already held everything this lens needed.
