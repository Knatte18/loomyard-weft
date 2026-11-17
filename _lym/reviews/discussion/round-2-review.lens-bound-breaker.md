```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked every bound, guard, guarantee and claim about existing code in `decision-record.md`.

**Claims attacked**
- D1: "`&lt;`, `&gt;` or `&amp;` is written as its unicode escape (`&lt;`, `&gt;`, `&amp;`)". **Broken as written.** The parenthesised escapes are the literal characters.
- D1: a `&lt;/script&gt;` body cannot end the script, given Go's default HTML escaping. Held.
- D2: rewriting identical bytes leaves no git change, and with one lym version `board.html` changes only when the binary ships a new viewer. Held.
- D3: the Bound for mixed versions, the downgrade removal and the cherry-pick conflict. Held. I found no counterexample within budget; the conflict way-forward path was not traced past the record.
- D4: the link allowlist (trim, tab/CR/LF removal, case-insensitive http/https, the `//` and `\\` rule) and the escape order keeping a target inside `href`. Held. I tried `java\tscript:`, `JaVaScRiPt:`, `&amp;#58;` entity smuggling (the pre-escape turns it into `&amp;amp;#58;`), ` //host`, `\/host` and `"onmouseover=`.
- D5: "so the verb runs from anywhere in a worktree", via `lymcwd.ResolveWithin`. **Broken** for a subpath-anchored worktree.
- D5: the refusal names "the worktree's anchored directory to run it from". **Broken** under `--board-path`.
- D5: "`--board-path` flag points the verb at that directory's `board.html`", given as following from the resolved layout. **Broken causation.**
- D5: the Bound on `xdg-open` running in the foreground, and the second key press. Held.
- D5: the opener spawn is logged and waited on (`PATTERN-child-processes`). Held as a design statement.
- D6: `"reveal": "silent"` brings the terminal forward on exit status 1. **Unevidenced.** It rests on VS Code behaviour with no source or version named.
- D6: `beam start` lives only in the workspace files, so a multi-root window holds exactly one. Held.
- D6: every caller of `RegenerateHubWorkspace` writes the members. Held: `ideengine` spawn, `loomcli` pawl wiring, fabric's after-verb path and anchorchange all run inside the lym binary, where root `PersistentPreRunE` installs the hooks.
- D6: the reserved-key guard covers "`folders`, `settings` or the beam file's generated mark". Held: `vscode.BuildWorkspace` writes only `folders` and `settings`, and `markBeamWorkspace` adds `lym.generated`.
- D6: a hook run inside the regeneration takes no lock, so the leaf lock still holds. Held: it reads only `ideengine.LymBinary`.
- D6 Bound: a git-tracked `.vscode/tasks.json` is skipped by seeding. Held (`internal/ideengine/spawn.go:246-255`).
- D6 Bound: an older binary rewrites the workspace files, `tasks.json` and the keys, and an up-to-date binary restores them on the next regeneration, seed or spawn. Held.
- D6 Bound: every multi-root copy of `board open` gives the same result. Held: a copy seeded by an older binary simply lacks the task, and every present copy resolves the same hub.
- Constraint: keys avoid `alt+shift+1` to `alt+shift+6`. Held.

**Findings**

1. **LOW · consistency · D5 first bullet (`decision-record.md:107-108`)**
   - **Summary:** "so the verb runs from anywhere in a worktree" overclaims.
   - **Counterexample:** in a worktree whose recorded anchor is a subdirectory, a terminal at the worktree root, or in any sibling directory outside the anchor, gets `ErrCwdOutsideAnchor` from `ResolveWithin` and the verb refuses.
   - **Fix:** say "from anywhere inside the anchored directory or its records sibling".
   - **Evidence:** `internal/lymcwd/lymcwd.go:136-160`, the `ResolveWithin` doc ("returns ErrCwdOutsideAnchor when cwd lies in a worktree but outside its anchored directory") and its `checkCwdWithinAnchor` call.

2. **LOW · consistency · D5 first and second bullets (`decision-record.md:109-114`)**
   - **Summary:** under `--board-path` there is no resolved layout, so the refusal's "anchored directory to run it from" does not exist.
   - **Further problem:** "From the resolved layout it … sets the board directory …, so the subtree's `--board-path` flag points the verb at that directory's `board.html`" gets the causation backwards. The flag bypasses the layout and config entirely.
   - **Counterexample:** `lym board open --board-path /tmp/x` with no `board.html` has `location == nil`, so the refusal has no directory to name.
   - **Fix:** state the flag's path as its own case, and say what the refusal names there (only `lym board rerender`, or the board path).
   - **Evidence:** `internal/boardcli/cli.go`, `PersistentPreRunE`: `if *boardPathFlag != "" { cfg = boardengine.Config{Path: *boardPathFlag} }` leaves `location` nil (cli.go:56 comment "nil under --board-path").

3. **LOW · design · D6 first bullet (`decision-record.md:131-133`)**
   - **Summary:** the claim that `"reveal": "silent"` brings the terminal forward when the verb exits with status 1 is a fact the design rests on, stated with no source or VS Code version.
   - **Why it matters:** VS Code's published task docs describe `silent` as revealing "only if the output is not scanned for errors and warnings". That wording differs from the exit-status rule the record states, so the visibility of D5's refusal is unproven.
   - **Fix:** cite the VS Code source or release that reveals a silent task on a non-zero exit, with its version, or choose `"reveal": "always"` with `"focus": false`.
   - **Evidence:** `decision-record.md:131-133`. The record cites nothing, and the code has no prior `silent` task to lean on: `internal/vscode/config.go` `writeAttachTask` uses `"reveal": "always"`.

4. **NIT · consistency · D1 (`decision-record.md:43`)**
   - **Summary:** "is written as its `&lt;`-style unicode escape (`&lt;`, `&gt;`, `&amp;`)" names the raw characters as their own escapes.
   - **Fix:** the escapes should read `<`, `>` and `&`.
   - **Evidence:** the raw bytes of line 43 (`cat -A` output above) show literal `&lt;`, `&gt;` and `&amp;`.

**Tool calls**
- `grep` of `internal/ideengine` for the tracked `tasks.json` skip, plus `vscode.BuildWorkspace` and `main.go`'s exits. The context lacked the skip site behind D6's Bound, the keys the reserved-key guard must cover, and the exit-code mapping.
- `sed` of `cmd/lym/main.go` `main` and `lymcwd.ownerOfRecordsSibling`. The context lacked the exit path for D6's status-1 claim and the records-sibling rule behind D5's "anywhere" claim.
