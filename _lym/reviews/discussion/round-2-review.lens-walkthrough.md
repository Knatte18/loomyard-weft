```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. Run each decision's mechanism end to end on one concrete case and report the steps that break.

**Walkthrough list**

- **D1**, newest binary boots a task session, then an older binary attaches it. The older binary stamps its build and re-pins `status-format[0]`/`[1]` and `status 2`. On the newest binary's next op, `bindingsStale` reads the older stamp as stale and `repinStaleBindingsLocked` re-pins. The session keeps `status 2` until its next attach, as the Bound says. **Purpose met.**
- **D2**, a hub-base `reed.yaml` holding `buttons: {task: {label: x, run: y}}`, a group written as a mapping. The load succeeds, but the record says it fails. **Claim broken** (finding 1).
- **D2**, a `label` holding `%`. Validation at render skips it with a Warn and the rest render. **Purpose met.**
- **D3**, a machine file holding `buttons.task: [{label: dl, run: …}]`. `LeafPaths` yields one unit, `buttons.task`. The template holds a sequence there, the shapes match, and `buttons` is covered by `CategoryMachine`, so it is admitted. **Purpose met.**
- **D3**, the same group written as a mapping in the machine file. `ValidateMachine` refuses `buttons.task.label` as not in the template, so the desktop load fails and adds no machine buttons, as D3 says. **Met for the machine layer.**
- **D4**, task `t1`, parent beam `b1` present, hub anchored at subpath `src`. The origin record has no `Beam`, so the place is `task`. `{worktree}` is `hub/t1/src`. `ParentOf` returns `"b1"`, and `NamedWorktree(hub,"b1").AnchorPath()` gives b1's own anchor; the presence check is a stat. Parent is the prime (`""`): the prime's anchor. **Purpose met.**
- **D5**, a click on `lymbtn1`:
  - `status-format[2]` draws the range from `#{E:}` of a session option, and `#{@strand}` selects the row.
  - The dispatch branch runs `run-shell -b -c '#{@lym_button_dir}'`.
  - With `@lym_button_dir` unset (a session rendered by an older binary), tmux falls back to home. The verb then finds no spec, shows a message and exits 0.

  **Purpose met.**
- **D6**, `{worktree}` = `/h/my hub/t1/src`. It is quoted as one word for the shell. Confirm text gets `#` and `%` doubled. `{strand}` is filled at click over a `run` whose static values are already in. **One edge broken** (finding 4).
- **D7**, CANCEL clicked in t1 with b1 present:
  1. The verb reads the spec, sees `confirm`, issues `confirm-before`, and exits 0.
  2. On `y`, it re-runs with `--confirmed`, and the hash matches.
  3. `lym batten cancel t1` runs in `hub/b1/src`. `Resolve` passes, since cwd equals the anchor. `ResolveBattenPrime(b1)` passes, since b1 is a beam that is not being landed.
  4. Cancel ends the processes and the session, sparing its ancestors (the verb and tmux). The message to the now-gone client fails at Debug, and the verb exits 0.

  Purpose met, but the rationale's premise is wrong (finding 3). The view-mode guarantee breaks on another path (finding 2).
- **D8**, the task set from b1's anchor, all met:
  - HALT: tender pause with no strand identity, so the operator is allowed.
  - `term`: `reed window` with an anchor dir, whose `reed attach` resolves; `--title` exists (`internal/reedcli/window.go:104`).
  - `agent`: `cd` into the task anchor, then `shuttle open --role agent`.
  - `code`: `ide spawn` by slug.

**Findings**

1. **MEDIUM — design — decision-record.md D2 (line 49) and the D2 acceptance criterion.** The claim "a group written as a mapping fails that config's load" is false for reed's base and worktree layers, and for desktop's hub file. Walking the load:
   - `fillMapping` checks only a mapping template node, or a scalar one against a mapping file node. A sequence template node has no case, so the mapping group is admitted (`internal/yamlengine/fill.go:111-124`).
   - `MissingKeys` skips `buttons.task[...]` paths once `buttons.task` exists (`internal/yamlengine/reconcile.go:177`).
   - `warnUnknownKeys` only Warns (`internal/configengine/config.go`, `warnUnknownKeys`).

   So `LoadConfig` succeeds and the render receives a mapping group. Only `ValidateMachine` refuses one, through `templateValueAt` returning nil (`internal/configengine/layers.go`, `ValidateMachine`). A plan following D2 would test for a load failure that never happens. Fix: say a mapping group loads with an unknown-keys Warn in reed's files and desktop's hub file and is skipped at render; only desktop's machine layer refuses it, failing that load.

2. **MEDIUM — design — decision-record.md D7 (lines 112, 115-116) and the D7 acceptance criterion "no click … opens a view-mode pane".** `run-shell` sees the exit status of the pinned shell line, not of the verb. The line names the absolute path `executablePath` returned in whichever lym process last pinned the bindings (`internal/reedengine/panebin.go:122-127`, `internal/reedengine/bindings.go:139-162`). When that binary is gone, every click exits 127 and tmux 3.6 opens view mode (D7's own probe). The stale re-pin does not help: it fires only on an older build stamp (`bindings.go:55-72`). Making the verb exit 0 on every path leaves this path open. Fix: the pinned line forces a 0 status itself (`; exit 0`, or the pwsh equivalent), or the record bounds this case.

3. **LOW — consistency — decision-record.md D7, line 113.** The rationale "CANCEL removes the task worktree" is false: `lym batten cancel &lt;slug&gt;` keeps the pair by default, and only `--remove` removes it (`internal/battencli/cancelcmd.go:56-57`). The run directory is still needed, because cancel runs only from the prime or a beam (`ResolveBattenPrime`), so only the stated reason is wrong. Fix: give that as the reason.

4. **LOW — design — decision-record.md D6, lines 101-105.** The fill runs in two phases: static values at render, then `{strand}` at click over the already-filled `run`. A static value that holds the literal text `{strand}`, such as a hub path containing it, is rewritten at click time. The record's "other brace text stays literal" fails there. Fix: fill `{strand}` only at the placeholder positions of the configured `run`, for example by keeping those positions in the spec or by filling all values in one pass at click.

Tool calls:
1. `grep` of `internal/reedcli/window.go` and `internal/shuttlecli`: checked that `--title` and `--role` exist for D8's commands. The manifest did not include those verbs.
