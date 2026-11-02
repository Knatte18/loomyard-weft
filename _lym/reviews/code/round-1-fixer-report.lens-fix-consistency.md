```yaml
lens: fix-consistency
outcome: reported
```

No findings.

- The new refusal row uses strand `ly:bm:orch` and goes through the same loop as its siblings. That loop asserts the error names the strand and contains "the orch of the entry's beam", which matches the refusal text in `refuseRunSettingsCaller`.
- The `ly:bm:orch-2` upsert sets `member` to profile `speed`, which is in the known-profile list (`docs, light, solid, speed, wide`).
- The upsert runs after the `get new` check and before the refusals loop. Each refusal subtest sets its own strand, so the leftover `ly:bm:orch-2` env does not leak into them.
- The `stepRunSettings` doc comment ("a beam's orch changes the entries of its own beam and no other") still describes the subtest accurately. No other doc or sibling test refers to the table's rows.
- Gate `lym gate test ./internal/boardcli` passed (exit 0), so both new cases behave as the finding describes.

I made no further tool calls; my context already held every changed file.
