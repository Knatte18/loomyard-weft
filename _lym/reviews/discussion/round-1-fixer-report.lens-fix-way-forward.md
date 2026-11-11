```yaml
lens: fix-way-forward
outcome: reported
```

I checked the ways forward the fix introduced, through the fix-way-forward lens: no findings.

- **Operator step in Scope Out:** switching the three keys with `lym config gate` from the prime can be carried out. The command exists, and `internal/configcli/doc.go:104` says it writes the `gate.yaml` of the checkout it runs in.
- **D5's unreadable-`Commit` miss:** it leads to the same whole run as today's `diff` scope, so it always makes progress.
- **D2's skip when a covered package is dirty:** a skipped package keeps its old map. D5 then runs that package whole until a clean prime rebuilds the map, so nothing gets stuck.
- **The new D4, D5, D2 and D7 criteria:** each can be built with a fixture module and hermetic git. That includes the base that predates or postdates a change on main, the two-`init` file, and the marker mismatch.

Tool calls:
- I searched `internal/` for the `lym config gate` command, because my context did not show whether it exists.
