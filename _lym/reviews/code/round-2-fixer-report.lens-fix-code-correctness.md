```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

The fix only edited `_lym/landing/summary.md`. It changed no code, so there was no function or branch to run and no caller or added test to check. I checked the new prose against the code instead:
- **Resolve-error handling.** `markUnresolvable` (`internal/configreg/configreg.go:135`) wraps only `ErrUnresolved` and `ErrNoRepository`, as the new bullet says. It returns every other error unchanged. The review confirmed that `applyReinterpretations` then writes no marker.
- **`read --envelope` assertions.** The new Tests bullet matches `cli_test.go:1089-1094`: `sending_repository` is asserted present, `Owner/xx-repo` on the first message, and absent on the second. `cli.go:327` emits the field only when it is set.
- **`hubs` row.** The new wording matches the subtest title at `cli_test.go:468` (a listed shortname, a hub with no repository and a forged name, each untrusted). It also lists the trusted GitHub-origin hub.

Tool calls:
1. `grep sending_repository` in `internal/creelcli/cli.go` and `cli_test.go`: my context lacked the envelope test assertions.
