```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each function and branch the diff adds or changes on one concrete input.

**The list**

1. `checkCwdAnchorGate`, when cwd equals the anchor (`home/user/repo`, `"."`): `samePath` is true, so it returns nil. Unchanged and correct.
2. `checkCwdAnchorGate`, from a subdirectory (cwd `/w/sub`, `"."`, `/w`): it returns the wrapped `ErrCwdOutsideAnchor`. The message names cwd, `/w` and `.lym-anchor`, the kept-cd cause, and `cd /w` with the rest of the way forward. Both `%s` slots get `anchorAbs` and the arguments line up. Correct.
3. `checkCwdAnchorGate`, with the worktree root above a subpath anchor (cwd `/w`, `"backend"`): the way forward reads `cd /w/backend`, the right target. Correct.
4. `checkCwdAnchorGate`, from a sibling of a subpath anchor (`/w/frontend`, `"backend"`): the message names `cd /w/backend`. Correct.
5. `checkCwdAnchorGate`, with an empty `anchorRel` (boundary): it can't happen, because `anchorRelOrRoot` only returns `"."` or a validated value. `filepath.Join(w, "")` would give `w` anyway. Correct.
6. `checkCwdAnchorGate`, with a path that holds a space (`anchorAbs` `/home/John Doe/hub/w`): the way forward reads `cd /home/John Doe/hub/w` with no quotes, so copying it as written fails (`cd: too many arguments` in bash). Finding 1.
7. `checkCwdAnchorGate`, from a symlinked cwd (macOS `/tmp` against `/private/tmp`): `workTreeRoot` comes from the resolved cwd, so `anchorAbs` is the real path and `cd` to it works. Correct.
8. `buildLocation` → `Resolve` → `preflight.ResolveMode` → creel `hubClient` → `runVerb`: for a wired hub's subdirectory, `ResolveMode` returns the gated error verbatim (`internal/preflight/predicates.go`, the `ErrCwdOutsideAnchor` branch). `hubClient` returns it unwrapped (`internal/creelcli/cli.go:219-221`), and `runVerb` passes it to `refuse(cmd, err)` (`internal/creelcli/cli.go:182-185`). Nothing replaces or swallows it. Correct.
9. `ResolveMode` from a plain repo's subdirectory: it degrades to standalone and the new text is never shown. Behaviour is unchanged by the diff, so this is not a finding.
10. `driftcli` `runCaptureChild`: `errors.Is(err, ErrCwdOutsideAnchor)` still holds, so the lasting-fault classification is unchanged and the longer message goes into the field. Correct.
11. `ResolveWithin` / `checkCwdWithinAnchor`: untouched; that refusal keeps its old text. Not a correctness issue.
12. `TestCheckCwdAnchorGate`'s new `wantMessage` check: it is guarded by `err == nil ||`, so a nil error can't panic. It is skipped on accept rows because it sits after the `!tt.wantErr` return. Correct.
13. `shipped.json` `lym-shell-directory`: the JSON is well formed, and order 30 puts it after order 20 and before the claude-settings entries at 100+. The `ShippedEntriesArePinned` order matches. Correct.

**Findings**

1. LOW, design, `internal/lymcwd/anchor.go:117`. The way forward prints `cd %s` with `anchorAbs` unquoted. For an anchored directory whose path holds a space, such as a Windows `C:\Users\John Doe\…` or any hub under a spaced directory, the command as written fails when pasted into a shell.
   - Evidence: `internal/lymcwd/anchor.go:115-118`, where the format string is `"way forward: cd %s and run the command again"` and the argument is a bare `anchorAbs`.
   - Fix: quote the path in the message, for example `cd %q` or explicit single quotes.

No other wrong result, swallowed error or inconsistent failure state turned up.

**Tool calls**

1. grep of `hubClient(` callers and `output.Err`: my context lacked how creel surfaces the `hubClient` error.
2. `sed` of `internal/creelcli/cli.go:176-192`: my context lacked how `runVerb` handles the error `hubClient` returns.
