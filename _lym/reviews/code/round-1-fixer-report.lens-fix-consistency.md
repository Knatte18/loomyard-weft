```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: no findings.

- **F2:** `procedureLenses["board-scope"] = true` (line 401) stays. `emphasisLineGaps(cfg.Lenses, procedureLenses)` at line 460 still uses it as the exemption set. The reworded error message agrees with the remaining condition.
- **F1:** `exempt` is used only inside the `emphasisLineGaps` subtest. `writerFanGaps` uses only its own locals.
- **Twins:** No sibling test uses the nested `emphasisLineGaps/writerFanGaps` name. The other `writerFanGaps` caller (line 406) is unaffected.

Tool calls: 1 (`git show --stat` plus two `grep` runs), to confirm that the other `procedureLenses` and `exempt` uses and the non-test sources do not depend on the removed condition.
