```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked every claim, bound and guarantee in the decision record and found 2 BLOCKING, 6 MEDIUM and 3 LOW.

**Findings**

1. **BLOCKING** — D6, D7, acceptance criterion D6: `{strand}` cannot be filled with "the full strand name in the clicking client's active pane".
   - The record plans to read the pane's `@strand` option, but `@strand` holds only the role label.
   - `setStrandPaneOptionsLocked` sets it to `strandLabel(s.Name)`, which returns `agentname.Parse(name).Role` (`internal/reedengine/paneoptions.go:18-29`).
   - Counterexample: an agent pane named `&lt;short&gt;:&lt;slug&gt;:discussion` gives `{strand}` = `discussion`, so any lym verb that takes a strand name gets a wrong target.
   - Reading `@strand` in Go would also break the code's stated rule that no Go decision reads these pane options (`paneoptions.go` header: "No Go decision reads them back").
   - The record must name a real source for the full name, such as reed state by pane id or a new decision-bearing pane option, and say which option becomes decision-bearing.

2. **BLOCKING** — D7, the confirm flow: the re-invoked line carries the filled `run` as an argument, and tmux mangles it.
   - `run-shell` format-expands its whole command before running it (tmux 3.6 manual: "Before being executed, shell-command is expanded using the rules specified in the FORMATS section").
   - `confirm-before` then parses the command again as a tmux command string.
   - Counterexample: `run: awk '{print $1}' f # note` loses or rewrites its `#` and `{...}` text during expansion. A `run` holding a single quote breaks the nested tmux quoting.
   - The comparison then fails ("buttons changed") or runs a different line.
   - `composeFollowCommand` doubles `#` for exactly this reason (`panebin.go:119-128`); the record states no such rule for the `run` text.
   - The confirmed path should pass a fingerprint, such as the index plus a hash of `run`, and read `run` from the spec again.

3. **MEDIUM** — D7 bound "a click reaches no other command": the spec lives in a session user option, and any process holding the tmux socket can rewrite it.
   - Every strand pane on the server has the socket (`TMUX` is set in-pane).
   - Counterexample: an agent pane runs `tmux set-option @&lt;spec option&gt; '&lt;json&gt;'` with a new `run` for an existing label. The drawn row, a separate option, still shows the old label. The operator's next click runs the injected line with the strand-name variables removed, so lym reads it as the external caller.
   - The Open risks line covers only a tracked `reed.yaml` edit, not this path. The bound needs stating, or the spec needs a home outside tmux, such as reed state under `.lym`.

4. **MEDIUM** — D7 "reads no working directory, config or reed state" contradicts "runs `run` through reed's configured shell".
   - The shell is reed config's `shell` key.
   - Either the spec carries the shell path, or the verb reads config, which this bullet rules out.

5. **MEDIUM** — D2: an invalid button fails every `lym reed up`, `lym reed resume` and add, which is a disproportionate blast radius.
   - Counterexample: one hub-base `reed.yaml` entry with `#` in a label. Every pair's `lym reed add` fails, so every run's agent spawn fails hub-wide, for a display convenience.
   - D3 handles the same invalid shape from the machine layer with a Warn and a skip, so the two sources behave differently for no stated reason.

6. **MEDIUM** — D1 and D5: older binaries still running break the third line.
   - The status-format and `status` pins have no build-stamp guard; only the bindings do (`bindingsStale`).
   - Counterexample 1: after the new binary pins, an older lym (a run pinned to its version, or a long-lived driver) boots a session or runs an attach pre-flight. It re-pins global `status-format[1]` with `follow`, so every session shows `follow` on lines 2 and 3.
   - Counterexample 2: that older lym sets `status 2` on the session it boots, which hides line 3 there.
   - Also, a live session booted by the old build keeps `status 2` and loses `follow` from line 2 the moment the new global `status-format[1]` is pinned, until its next attach.
   - The record is silent on all three; the existing stale-stamp mechanism could cover the status pins.

7. **MEDIUM** — D7: the verb's trace file may not land in the worktree's log directory.
   - The claim is "the verb running in the worktree so the trace file lands in its `.lym/logs/`".
   - `run-shell`'s working directory is the server's unless `-c` is given, and the global binding knows no worktree.
   - If the verb changes directory itself, any log line written before that, through the logger's lazy `LogsDir(lymcwd…)`, lands under the server's starting directory.
   - The record should state the mechanism: `run-shell -c '#{session_path}'`, which equals the anchor, or a working-directory change before the first log line.

8. **MEDIUM** — D4: the lazy closure runs at every attach pre-flight under reed's op lock, and for a task it calls `hubgeom.ParentOf`.
   - `ParentOf` lists worktrees through `fabricengine.List`, a git spawn, and needs the prime's location, not the task's.
   - Counterexample: an attach while git is slow or locked holds the op lock behind a git call that only fills `{beam}`.
   - The record should state the prime-location resolution and accept or bound this cost. Only `CANCEL` uses `{beam}`, so it could also be resolved at click time.

9. **LOW** — D3: the machine layer's "shape must match the template" check against the template value `task: []` is unverified for a non-empty list of maps.
   - The record asserts it in acceptance criterion D3 and settles nothing about it.
   - `lym config desktop --set` likely cannot write a list of maps either, which leaves the operator a hand edit of the machine file.

10. **LOW** — D8, CANCEL: `lym batten cancel` ends the pair's strands and may take down the session the click came from.
    - The verb survives as a child of `run-shell`, not a pane, but its failure `display-message -c &lt;client&gt;` may target a client that has gone. That is harmless.
    - A `cancel --remove` added later would delete the worktree the verb runs in. The record should state that the verb tolerates its own working directory disappearing.

11. **LOW** — D5: the spec option and the two row options are written by separate `set-option` calls.
    - A click between the writes during an attach can hit a new spec under the old row.
    - The confirm re-check guards only `confirm` entries; a plain entry would run the new command at that index.
    - The window is narrow but contradicts "a click always runs the command drawn under it".

**Claims attacked**

- Global `status-format` array, and that per-session indexing hides lines: held.
- User range name of at most 15 bytes (`lymbtn&lt;N&gt;`): held.
- Ranges coming out of `#{E:}` are drawn and clickable (expand, then draw): held.
- tmux 3.4 floor covers `#{m:}`, `confirm-before -p`, `run-shell -b`, `display-message -c`: held.
- `#{?#{@strand},…}` evaluates against the client's current pane: held.
- `{strand}` is the full strand name from `@strand`: broken (1).
- Confirmed re-run compares an unchanged `run`: broken (2).
- Only config-sourced commands reach a click: broken (3).
- The verb reads no config: broken (4).
- Validation failing reed up and add is safe: broken in blast radius (5).
- `follow` alone on line 3 and `status 3`: broken under older binaries (6).
- Trace file lands in the worktree: unbounded (7).
- Building the geometry stays read-free while the closure stays cheap: partly held (8).
- Machine `buttons` loads through the shape check: unverified (9).
- Strand-name variables removed lets lym verbs read the click as the external caller: held, since the tmux server environment is scrubbed of engine keys only.
- Batten verbs run only from the prime or a beam, hence `cd {beam}`: held.
- `lym ide spawn` accepts the prime's name: held.
- Row and spec stay consistent: broken narrowly (11).

**Tool calls**

- `paneoptions.go` grep for `@strand` plus a `LeafPaths` grep: the context did not show what value `@strand` holds.
- `man tmux` `run-shell` plus a grep of agent settings for tmux denies: the context lacked `run-shell`'s format-expansion rule.
- `strandLabel` body plus the `run-shell` man entry: needed to confirm `@strand` holds the role only, and the expansion wording.
