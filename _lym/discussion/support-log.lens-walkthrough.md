```yaml
lens: walkthrough
outcome: reported
```

Task: a walkthrough-lens review of the reed-status-buttons decision record. I ran each decision's mechanism on one concrete case and checked each step's input against what the step before it produces.

**Findings**

1. **BLOCKING — D6 / D7: `{strand}` cannot be filled with the full strand name from `@strand`.**
   Case: the operator clicks a `window: agents` button whose run is `lym creel post {strand} …`, with the active pane on the orch strand. D7 fills `{strand}` from the active pane's `@strand`. `setStrandPaneOptionsLocked` sets `@strand` to `strandLabel(s.Name)`, and `strandLabel` returns `agentname.Parse(name).Role`, which is the role only (`internal/reedengine/paneoptions.go`). The command gets `orch` where D6 promises the full strand name, so `{strand}` misses its purpose of naming the agent in the active pane.
   Fix: put the full name in a pane option of its own, or say exactly where the full name is read from.

2. **MEDIUM — D7: the verb needs "reed's configured shell" but reads no config.**
   Case: a click on HALT. D7 says `lym reed button` reads no working directory, config or reed state, only the spec option and the client's pane. The next step runs `run` "through reed's configured shell", which is `Config.Shell` in `reed.yaml`. Nothing the verb is handed carries it. The spec (D5) holds index, label, run, confirm and worktree, with no shell.
   Fix: add the shell to the spec, or pin it into the click line the way the socket and tmux paths are pinned.

3. **MEDIUM — D7: the trace file has no route into the worktree's `.lym/logs/`.**
   Case: CANCEL fails. D7 says the verb runs in the worktree so its trace file lands in that worktree's `.lym/logs/`. The binding runs `run-shell -b` with no `-c`, so the process starts in tmux's default directory, not the worktree. The logger's durable sink resolves `LogsDir` from the process cwd when the first record is written, and the root hook's trace setup runs before the verb body. The verb reads no working directory, so no step moves it into the worktree. The output log, which is half the stated purpose, lands in some other directory or none.
   Fix: say how the verb gets there, either `run-shell -c '#{session_path}'` in the branch or a chdir to the spec's worktree before any log record. Also check that `session_path` is the anchor.

4. **MEDIUM — D7: `--confirmed` compares a `{strand}`-filled run against the spec's unfilled run.**
   Case: a `window: agents` entry with `confirm`. The first invocation fills `{strand}` at click time and passes "the entry's filled `run`" through `confirm-before`. The confirmed invocation checks that the spec at that index "still holds that `run`". The spec holds only the static fill (D5/D6), so the strings never match. Every confirmed `{strand}` button would refuse with "buttons changed".
   Fix: compare against the run before the `{strand}` fill, and fill `{strand}` again, or pass it separately.

5. **MEDIUM — D7: `run-shell` output pops a view-mode pane.**
   Case: any click. The tmux 3.6 manual says that without `-C`, any stdout of the `run-shell` command "is displayed in view mode … after the command finishes". If `lym reed button` prints the usual JSON envelope, as lym verbs do through `internal/output`, every click covers the active pane with that envelope. The record does not say the verb writes nothing to stdout, or that the line redirects it. The follow button uses the same `run-shell -b` form, and I could not confirm within my budget how it avoids this.
   Fix: state the bound, e.g. the verb prints nothing or the pinned line discards stdout.

6. **LOW — D4: `ParentOf` needs the prime's location and returns a slug, not a path.**
   Case: the task `reed-status-buttons` under the prime. `hubgeom.ParentOf(prime *lymcwd.Location, slug)` takes the prime's location, but `ReedGeometry` holds the task's `l`. It returns a slug, `""` for the prime, so getting `{beam}`'s absolute path takes two more steps: resolve the prime location, then map slug to worktree path. Both are workable, but the record's "the pair `ParentOf` names" skips the slug-to-path step.

7. **LOW — D5: the attach pre-flight now spawns git under the op lock.**
   `ParentOf` calls `fabricengine.List`, a worktree listing, from inside the render closure. That closure runs in `pinGeometryOptionsLocked` at every attach, under the op lock. This is a cost the record does not mention; the purpose still holds.

**Decisions, case used, purpose met or not**
- D1 (task session booted: status 3, global `status-format[2]`, follow first): met.
- D2 (invalid label with `#` fails `lym reed up` through `validateWindowConfig`, which `lifecycle.go`, `strand.go`, `replace.go` and `ownwindow.go` call): met.
- D3 (machine `task` entry appended after reed's task group): met. Setting a list through `lym config --set` is not covered, which does not touch the purpose.
- D4 (task with the prime as parent gets `{beam}` = prime path): met, with finding 6.
- D5 (agent window vs overview through `#{?#{@strand},…}`): met for row selection. The spec content feeds findings 2 and 4.
- D6 (path with spaces quoted as one word): met for the static placeholders. Not met for `{strand}` (finding 1).
- D7 (HALT click, CANCEL confirm, failing command): not met (findings 2, 3, 4, 5).
- D8 (CANCEL is `cd '&lt;beam&gt;' &amp;&amp; lym batten cancel &lt;slug&gt;`, which passes `ResolveBattenPrime` from the beam): met.

**Tool calls**
- Grepped `@strand` in `paneoptions.go`, the `validateWindowConfig` call sites, the `run-shell` man entry and the reed pre-run guard: my context lacked what `@strand` holds, where validation runs, and `run-shell`'s cwd and output behaviour.
- Read `strandLabel` and the full `run-shell` man text: needed to confirm `@strand` holds only the role and that stdout goes to view mode.
