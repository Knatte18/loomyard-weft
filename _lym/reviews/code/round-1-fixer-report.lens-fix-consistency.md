```yaml
lens: fix-consistency
outcome: reported
```

Task: checked that the fix change set (F1–F5, the smoke test and the landing summary) still agrees with its surroundings.

1. **LOW**: `internal/loomcli/smoke_drivingsurface_test.go`, the new comment above subtest "loom resume leaves a follow-off session off".
   - The added line "They also leave a status strand with no sidecar, which the resume replaces." repeats the older comment a few lines below, "The resume replaces the status strand added above, so that strand's window does not."
   - One fact is now stated twice, so dropping one of the two keeps them from drifting apart.

I found no other disagreement: the remaining places I checked in `internal/loomcli` and `docs/overview.md` still match the change.

Tool calls:
- grep over `internal/loomcli`, `internal/statuspane`, `replace.go` and `docs` for status-strand and caller wording: my context held no repo-wide search for twins of the changed text.
- `sed -n` of `docs/overview.md` 1196–1225 and a grep of `cli.go`/`start.go` for resume descriptions: my context did not hold the overview's resume passage.
