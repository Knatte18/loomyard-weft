```yaml
lens: fix-proposal
outcome: reported
```

No findings.

- **F1:** the moved block uses only its own locals; both subtests call `t.Parallel()` first; nothing depended on the nesting; the subtest name changes from `emphasisLineGaps/writerFanGaps` to `writerFanGaps` and nothing pins the old name.
- **F2:** `procedureLenses["board-scope"] = true` stays because `emphasisLineGaps` reads it as the exemption. The dropped half could never fail, so no coverage is lost.

No further tool calls were made.
