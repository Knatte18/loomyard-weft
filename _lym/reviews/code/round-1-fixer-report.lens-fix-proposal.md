```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals carried out as proposed (F2, F3, F4, F6, F7).

Findings:

1. **LOW**, `_lym/landing/summary.md`, the F3 lines under `## Hub config` ("The stored maps stay on disk with no reader. Delete ... by hand."). F3's proposal contradicts the cross-binary paragraph that F2 added under `## Merge danger`. That paragraph says older binaries may still be running with the report steps on. Those binaries still read and write `.lym/testcover`: at `main`, `internal/testreach/reader.go:75,121` reads the map directories, and `internal/testcover` `Dir` names them. So "with no reader" is false until every binary is updated, and maps deleted earlier are rebuilt by an older binary. The sentence should say to delete them once every binary of the hub is updated.

No other findings:
- **F2:** the step can be carried out. `gates` is a per-name map, and `setPerNameEntry` (`internal/yamlengine/set.go:646`) takes a whole flow mapping with no per-key shape check, so `reports: []` can be written. I did not run it, and did not read `perNameEntryNode` or `sameValue`.
- **F4:** `exportTierSelection(tags)` runs in `main` ahead of the modes (`cmd/testtiming/main.go:164`), and `-redundancy` goes through `withGateSlot`.
- **F6 and F7:** no breaking case found.

Tool calls:
1. grep of `internal/gateslot/config.go` for the config spec: my context did not show whether `gates` is an open or per-name map.
2. grep of `cmd/testtiming` and `main:internal/testcover` and `testreach`: the redundancy mode's environment and the map paths and readers were not in context.
3. grep of `internal/yamlengine/set.go`: the `--set` behaviour for a whole entry was only partly in context.
4. grep for `exportTierSelection`: its call site was not in context.
5. grep of `setPerNameEntry` and `dropTemplateNames`: their bodies were not in context.
