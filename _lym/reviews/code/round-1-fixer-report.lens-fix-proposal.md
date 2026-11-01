```yaml
lens: fix-proposal
outcome: reported
```

**Findings**

1. **MEDIUM**, `docs/overview.md`, "Its `on_land` action runs once it lands or its entry is removed": `fronttask.Remove` drops the record before it removes the entry, so a removal through `Remove` runs no action; only a bare `lym board remove` leaves the record standing. Suggested: write "once it lands" only, or name the bare-removal case.
2. **LOW**, `internal/adoptcli/green.go`, `resumeHaltedChildren` after the F4 reorder: `deps.ParentOf` now runs for every non-done task before the subject filter, so a parent read failure of another beam child still adds a line; probably unavoidable, name it in the report.
3. **LOW**, `internal/fronttask/hub_integration_test.go`: only the prime owner is covered; the `beam != ""` branch is untested. Suggested: add a beam row.
4. **LOW**, `internal/fronttask/fronttask.go`, `File` returning nil on a failed record write: if the entry lands or is removed before any `Settle` records it, the action never runs. Defensible; note it as a trade-off.

No problems with the F1 seam, the F3 reason text or the test use of the global logger.

Tool calls: none.
