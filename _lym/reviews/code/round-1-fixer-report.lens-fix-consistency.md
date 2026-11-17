```yaml
lens: fix-consistency
outcome: reported
```

**Task:** check the fix change set for stale twins and sibling disagreements (fix-consistency lens).

Findings:

1. **LOW**, `internal/gateslot/doc.go:176-180`. The `redundancy_test_memory_mb` paragraph says what the cap does and that a capped mutant run is a kill. It does not mention that both `lym gate testmap` and `lym gate redundant` now run fewer tests side by side when memory is short.
   - `internal/gateslot/template.yaml` line 6 now says so for both verbs.
   - Evidence: the `template.yaml` edit and the `gatecli/doc.go` edits state the clamp. The gateslot package doc still describes only the cap.
   - Nothing in it is false, so this is a gap, not a contradiction.

I found no other stale twin:
- The `lym gate testmap` help text, the `gatecli/doc.go` testmap paragraph (`:1057-1058`) and the `docs/overview.md` testmap sentence all state the clamp consistently.
- `doc.go:1069` ("recorded as failed") agrees with the new testmap test.
- The `template.yaml` wording (left out, recorded as failed by testmap, kill in redundant) matches `internal/gatecli/doc.go:590`, `:1069` and `:1148`.
- The stale "Nothing else builds the maps" sentence and the old `gate redundant:` log prefix appear nowhere else in `.go` or `.md` files.
- Every `testMapRequest` literal now sets `cappedParallel`: the CLI constructor and the three test sites.
- The `RunMemoryCapped` doc, the `internal/proc/doc.go` sentence and the `spawnobservability_test.go` reason all say the probe spawn is unlogged and its outcome is logged through `MemoryCapCoversTree`.

Tool calls (1 of 6 used):
1. One `Bash` call: `sed` of `internal/gateslot/doc.go` 170-182, `grep` of `internal/gatecli/redundant.go` for side-by-side and memory text, a repo `grep` for the old log prefix and the removed help sentence, and `sed` of `internal/gatecli/doc.go` 1066-1072 and 1146-1162. My context lacked the gateslot package doc and the remaining `doc.go` spans.
