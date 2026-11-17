```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals the fixer carried out as proposed (F1 to F4) for a case where the change is wrong or breaks something elsewhere.

No findings.

- **F1:** the `--label bug` assertion matches the code. Only `soon` carries `bug`, and the root row is gated off by `len(listLabels) == 0`. Dropping that guard would print a P0 row and fail the exact-string check.
- **F2:** `BriefTask.IsDone` tests `Layer == doneLayer`. `ListTasksBrief` fills `Layer` from `ComputeLayers`, which assigns `doneLayer` to done entries, so the predicate sees what `RenderOrder` sees. The only way the predicate misses is when layer computation fails and every `Layer` is empty, so the row then falls back to the old placement.
- **F2 test:** the all-done case checks four lines with P0 at index 1. That fails if the `IsDone` clause is removed, because the row would then land at the end.
- **F3:** appending the root row after `sortByDate` is safe. `RenderCompact` routes sketches into their own trailing block, so the P0 row still precedes the `sketches:` heading. The append only affects the `--text` path, because the JSON path returns earlier. The new `--sort age` assertion checks the last line is the P0 row.
- **F4:** the doc comment now matches the assertions, which expect `boardengine.RootSubject`.

I made no tool calls.
