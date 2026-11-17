```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged each proposal the fix carried out as written.

**Proposals challenged:** F1, F2, F3, F4, F5, F6, F9, F10, F11, F12, F13, F14, F15. F7 and F8 are decision findings, so the fixer did not apply them.

1. **MEDIUM. D4 bullet 5 (F3): the allowlist still lets a `/\host` target through.**
   - The proposed rule keeps any target that has no scheme and does not start with `//` or a backslash.
   - A target `/\host/share` passes that test. Browsers parse URLs by the WHATWG standard, which treats a backslash as a slash for special schemes. `file` is a special scheme, so from a page opened at `file://` the target resolves exactly as `//host/share` does.
   - The same gap applies to an embedded tab or newline such as `java<TAB>script:alert(1)`. The rule trims leading characters only, while the URL parser strips tab and newline anywhere. A naive scheme test sees no valid scheme in `java\tscript` and keeps the link, and the browser then runs it as `javascript:`.
   - The rule should strip tab, CR and LF throughout the target, and refuse a scheme-less target whose first two characters are any mix of `/` and `\`.

2. **MEDIUM. D3 Bound (F13): the downgrade conflict is the expected outcome and has no way forward.**
   - The older binary's sync commit removes both board files (`board.html`, `board-data.js`) and also rewrites `README.md` and the project files.
   - Because the commit touches unclaimed paths, the whole commit is cherry-picked (`replay.go:742`). Its `README.md` change therefore loses the by-entry merge too.
   - `board-data.js` changes on every board write by a newer machine. So the cherry-pick meets a modify/delete conflict whenever another machine has written the board since, which is the usual case.
   - `replayBolt` then returns `ErrReplayConflict` with the starting tip restored (`replay.go:702-703`). The bound says "can conflict" but gives no recovery, and the next write by the older binary repeats the same conflict.

3. **LOW. D6 Bound (F9): a window seeded before the upgrade is now listed under `alt+shift+0` only.**
   - After the split by key, that clause sits in the `alt+shift+0` sentence.
   - `board open` lives in `tasks.json`, which only a seed or spawn writes (`vscode.WriteConfig`). A window seeded before the upgrade therefore also lacks it, so `alt+shift+9` misses there too.
   - The rewritten `alt+shift+9` sentence names only the tracked `tasks.json` case.

4. **LOW. D6 bullet 1 (F6): `"reveal": "silent"` is described as bringing the terminal forward on any non-zero exit.**
   - As I recall VS Code's task runner (`terminalTaskSystem`), it reveals the terminal only when the exit code is exactly 1 and no problem-matcher markers were found, not on every non-zero exit. I did not check the VS Code source, so this is unconfirmed.
   - lym's refusals exit with 1 (`clihelp.Abort(ctx, 1)`, `internal/boardcli/cli.go:142`), so the behaviour holds for D5. The sentence should say exit 1 rather than any non-zero exit.

5. **LOW. D6 seam bullet (F1): the general seam does not say what happens when a hook returns a reserved key.**
   - The hook returns top-level members keyed by name, and fabric splices them verbatim.
   - Nothing stops a member named `folders`, `settings` or `lym.generated`. Splicing one would write a duplicate JSON key into the workspace file.
   - A general seam should state the rule: reject or skip those keys.

Outside this lens: the added D6 criterion says a regeneration with no hook installed writes files holding only `folders` and `settings`. A beam's workspace file also carries the `lym.generated` mark (`markBeamWorkspace`, `hubworkspace.go:227-230`), so that criterion fails for beam files.

The other proposals held against the cases I built:
- **F2:** the `ResolveWithin` change works because `LoadConfig` only needs the layout.
- **F4:** the `LymBinary` fallback matches `writeWorktreeConfig`.
- **F5:** the mixed-version bound holds.
- **F10, F11, F12 and F15:** these hold.
- **F14:** Go's `encoding/json` writes `<`, `>` and `&` by default, as the fixed text now says.

Calls made:
- Read `cmd/lym/main.go:135-160`. My context lacked the condition guarding the hook installs, and I needed it to check whether a production regeneration could run with no hook installed. It cannot: the guard is `!testing.Testing()`.
