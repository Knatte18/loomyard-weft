```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked the record's decisions pairwise, then each criterion and constraint against the decisions it touches.

**Pairs checked**
- D1 × D2 (`board.html` loads `board-data.js` through a script tag): held.
- D1 × D3 (`board-data.js` carries a render time, rewritten on every write and committed with README): held, since README ages also change on every render.
- D1 × D4 (the viewer gets the fields it needs: kinds, labels, section, parent handle, render time, owner): held.
- D1 × D4 (D4 orders sections "in the README's order"; D1 says the viewer "recomputes nothing"): broken, see finding 4.
- D1 × D4 (link targets: D1 gives design and decision names "when they exist"; D4 links "every entry to its design file"): broken, see finding 5.
- D1 × D7 (`<details>` wrapping leaves the section placement D1's criterion compares against): held.
- D1 escaping × D4 escaping (JSON-level escape, then the HTML-escape the viewer applies after parse): held.
- D2 × D3 (identical bytes give no git change; manifest removal on downgrade): held, the bound is stated.
- D2/D3 × D5 (refusal → `lym board rerender` → `Board.Rerender` → `RenderToDisk` writes `board.html`): held.
- D5 × D5 and D5 criteria (`ResolveWithin` against "reads the board directory as the other board verbs do"): broken, see finding 1.
- D5 refusal × D6 `board open` presentation (terminal not revealed): broken, see finding 2.
- D5 xdg-open bound × D6 key (blocking verb, key pressed again): broken, see finding 6.
- D6 `board open` in `tasks.json` × D5 cwd (task cwd is the anchor folder): held.
- D6 `beam start` in workspace files × D6 bound list: broken, see finding 3.
- D6 keys × existing `ctrl+alt+a` and the mill-keys constraint: held.
- D6 × Scope ("written by `lym ide seed` and `lym ide spawn`" against D6's `RegenerateHubWorkspace`, which fabric verbs also call): broken, see finding 7.
- D3 replay × D7: held.
- Criteria and constraints:
  - D4 `Review:` × the no-JS-runtime constraint: held.
  - The D1 `</script>` criterion × D1: held.
  - D6 criteria × the tracked-`tasks.json` skip: held.
  - `PATTERN-test-only-code` × the D5 opener seam: held.

**Findings**

1. **MEDIUM** · design · D5 and the D5 subdirectory criterion. D5 says the verb resolves with `lymcwd.ResolveWithin` and also says it reads the board directory "as the other board verbs do". The two contradict each other: the board subtree's `PersistentPreRunE` runs the strict `lymcwd.Resolve` for every subcommand except `board` and `intake`, and it aborts on a subdirectory cwd before `open`'s RunE runs. The criterion "run from a subdirectory … hands the opener seam the absolute path" cannot pass unless D5 also states that `open` is exempted from that pre-run, or that it changes the pre-run. Evidence: `internal/boardcli/cli.go:124-148`.

2. **MEDIUM** · design · D5 refusal × D6 `board open` presentation. D6's `board open` task "does not reveal or focus the terminal", so pressing `alt+shift+9` on a hub with no `board.html` (or with a failing opener) shows nothing. D5's refusal and its way forward, `lym board rerender`, go to a terminal the operator never sees. Neither decision bounds this: no reveal on failure is stated, and no bound says the refusal is invisible from the key. Evidence: decision-record.md D5 bullets 2 and 4, D6 bullet 1.

3. **LOW** · consistency · D6 bound. The bound puts every missing-task window under `alt+shift+0`, including "any window whose worktree has a git-tracked `.vscode/tasks.json`". But `beam start` lives in the workspace files, which a tracked folder `tasks.json` does not affect. What a tracked `tasks.json` drops is `board open`, which is `alt+shift+9`. Seeding skips a tracked file and logs a warning (`internal/ideengine/spawn.go:246-255`). The bound misnames the key, and no bound for `alt+shift+9` exists.

4. **LOW** · design · D1 × D4 section order. D4 groups a project's entries "by section in the README's order". D1 gives each entry a section name plus only the type-label order: no promise that the entry list is in README order, and no order for Running, Ready, the layers, Independent, Draft and Done. That leaves the viewer to recompute the order, which contradicts D1's "the viewer recomputes nothing". Evidence: decision-record.md D1 lines 33-38 and D4 line 78.

5. **LOW** · consistency · D4 bullet 4 × D1. D4 links "every entry to its design file", but D1 carries design file names only "when they exist", and a design file exists only for an entry with a body (`internal/boardengine/render.go:2`). D4 should say "when D1 names one".

6. **LOW** · design · D5 bound × D6 key. When `xdg-open` runs a browser in the foreground, the `board open` task stays active until that browser exits. A second `alt+shift+9` then gets VS Code's "task is already active" prompt rather than opening the board. D5's bound covers the wait but not this effect through D6's key. Evidence: decision-record.md D5 bound at line 102 and D6 line 117.

7. **LOW** · consistency · Scope × D6 and the Notes. Scope says the tasks and keys are "written by `lym ide seed` and `lym ide spawn`". D6 and the Notes put `beam start` in `fabricengine.RegenerateHubWorkspace`, which the fabric verbs (`regenerateAfterVerb`), `anchorchange.go:539` and `loomcli/pawlwiring.go:116` also call. Evidence: decision-record.md line 15 against lines 115 and 195-196.

**Tool calls**
- `sed -n 120,175p internal/boardcli/cli.go`: read the board subtree's `PersistentPreRunE` cwd resolution. My context had the `--board-path` lines but not the `Resolve` call the D5 pair turns on.
