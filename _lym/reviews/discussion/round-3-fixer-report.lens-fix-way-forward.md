```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that every way forward the round-3 fix added can be carried out and makes progress.

The new ways forward are:
- **D1 and D4 scans:** both scans and the producer table now skip a spec literal that sets no `Prompt`.
- **Bouncer prompt render:** a producer whose launching function fills its prompt inline gets that render split out, as `judgeCall` and `runSeedSpawn` need.
- **D12 non-exec payload:** the nudge returns no nudge and leaves the rollout unread.
- **D12 `function_call`:** it ends a `lookup_run` or `edit_run` chain.
- **D12 read-again rule:** both paths are resolved against a `cwd`, and the earlier read must be no older than the file's last change.
- **Acceptance criteria:** the new D1 and D12 rows.

Each one can be carried out:
- **Scans:** checking whether a composite literal sets `Prompt` can be done statically. The tackler and arbiter attach paths go through builder functions whose literals set `Prompt` from a parameter, so the scans still cover those builders as launch literals.
- **Render split:** this is a code change inside the producer's own package, and nothing blocks it.
- **Read-again rule:** every input it needs is in the cited data. The payload carries `cwd` (spike `payloads.jsonl`), the `CommandExecution` item carries `cwd` as a `file://` URL, and each rollout line has a `timestamp`.
- **`function_call` rule and its criterion:** a fixture rollout of lookup cells, then a `spawn_agent` `function_call`, then a lookup cell tests both. The nudge fires only on the payloads of the nested calls around the `function_call`, so the test reaches the rule.
- **Non-exec payload:** the rule holds whether or not `PostToolUse` fires for a collaboration call, so it needs no spike first.

no findings

No further tool calls made.
