```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1–F5 change set for twins or sibling text left stale (fix-consistency lens).

**LOW — F5's twin call sites were left as they were.**
- **Changed text:** `internal/loomengine/seed.go`, where the `os.MkdirAll(filepath.Dir(statusLockPath))` call before `state.ReadJSON` was deleted.
- **Summary:** The same mkdir-then-`ReadJSON` pattern remains at five other sites. Each calls `MkdirAll` on the lock directory just before a `ReadJSON` call.
- **Evidence:**
  - `internal/beamcli/deps.go:96`
  - `internal/battencli/startguard.go:297`
  - `internal/loomcli/circling.go:222`
  - `internal/hibernatecli/deps.go:128`
  - `internal/tendercli/deps.go:297` and `:607`
- **Contradiction:** `internal/state/doc.go:50` says "ReadJSON makes none" of the directories, but these callers still make the lock directory themselves before reading.
- **Why LOW:** The review scoped F5 to `VerifySeedOwnership` and rated those callers unreachable from the hub reader. The fix is consistent with the review's text, but the stated rule is now only half applied.
- **Not checked:** Whether those sites run in a hub-reader path, so I can't say they need changing.

Other siblings agree with the change:
- **`internal/state/doc.go:50` and `ReadJSON`'s doc comment:** both state that a read creates no directory, which matches the F1, F3 and F5 tests.
- **`HubOrchs` doc comment:** it promises that a pair still being created is "skipped silently", which matches the F4 assertion on `level=WARN`.
- **`add_hubreader_integration_test.go` header:** it now covers `ReadOrigin`, as the new test does.
- **`VerifySeedOwnership`'s doc comment:** it never mentioned the mkdir, so nothing there went stale.

Tool calls:
1. `grep` for `MkdirAll` near `ReadJSON` across `internal`, to find twin call sites. My context lacked those files.
2. `grep` for `VerifySeedOwnership` in `.go` and `.md` files, to find its callers and docs. My context lacked them.
3. `grep` of `internal/state/doc.go` for "creates no directory", to check the stated rule. My context lacked that file.
