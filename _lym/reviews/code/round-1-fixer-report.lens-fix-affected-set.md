```yaml
lens: fix-affected-set
outcome: reported
```

Fix-affected-set lens: I reran the search for the set behind F1, the sites that register a fixer pane process. F2 and F3 have no set premise.

Findings: none.

Unchecked: F2 (a single-key test row) and F3 (the one test table), because neither premise is a set.

F1: `Registrar` has three callers, and `fixerTimeout` covers all of them.
- `internal/loomcli/gatefixer.go:228` and `internal/gatecli/cardfixer.go:175` register `gatefixer.RoleGateFixer`. They take the gate config's `FixerTimeout()`, which the strand is spawned with.
- `internal/webstercli/fixverify.go:42` registers `gatefixer.RoleVerifyFixer`. It takes webster's `RecoveryTimeoutMin`, the bound its spawn uses.
- `gateProcessLive` has one caller, `internal/battencli/wire.go:680`, already updated to the new signature. All other `ChildRecord` sites use non-fixer roles, which `IsFixerRole` skips.

Tool calls:
1. `grep` for `gatefixer.Registrar(`, `gateProcessLive(` and `ChildRecord(` across non-test Go files. I used it to rerun F1's set of fixer-registering and caller sites. My context held only `Registrar`'s definition, not its callers.
