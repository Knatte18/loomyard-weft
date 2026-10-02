# Card 10 — pane-title-mirror

**Edit:**
- `internal/reedengine/spawn.go`
- `internal/reedengine/overlay.go`
- `internal/reedengine/parse.go`
- `internal/reedengine/doc.go`
- `internal/reedengine/parse_test.go`
- `internal/reedengine/spawn_test.go`

**Create:**
- `internal/reedengine/naming_integration_test.go`
- `internal/reedcli/naming_integration_test.go`
- `internal/standalonegeom/naming_integration_test.go`

**Uses:**
- `internal/agentname/agentname.go`
- `internal/reedengine/panebin.go`
- `internal/standalonegeom/reedgeom.go`
- `internal/standalonestate/standalonestate.go`

**Intent:**
Mirror each strand's full name into its pane title, and stop the agent inside from overwriting it.
In `launchStrandLocked`, right after `split-window` validates the new pane and before the launch line is typed, reed runs `set-option -p -t <pane> allow-set-title off` and then `select-pane -t <pane> -T <full name>`.
Both are display mirrors: no lookup or delivery ever resolves through a title, and strand state stays the only key.
A failure of either is logged with a named `logger.Warn` and does not fail the launch, since a missing title costs display only.

`listPanes` appends `#{pane_title}` as the last format field and `parsePaneList` reads everything after the sixth field as `LivePane.Title`, so a title holding spaces survives; the doc's list-panes format line moves with it.

Tests: `parse_test.go` parses a title with spaces and an empty title; `spawn_test.go` asserts, on the fake tmux, the two commands in order between the split and the send-keys, carrying the strand's full name.

`naming_integration_test.go` (`integration` tag, real tmux, built like the package's other integration engines with a told code and slug): an added strand's state name and pane title are both `<code>:<slug>:<role>`, an empty slug gives `<code>:<role>`, `show-options -p -v allow-set-title` on the pane reads `off`, and a launched `sh` command writes `$LYX_STRAND_NAME` and `$LYX_PARENT` to a file the test reads back.
`reedcli/naming_integration_test.go` (`integration` tag) on a `hubforge` hub with a task worktree whose default-run seed (written through `shedrun.WriteSeed` with a `Parent`) records a parent: `lyx reed add --role worker` twice yields `worker` and `worker-2` in `lyx reed status`, the launched process sees `LYX_PARENT`, and after `tmux kill-server` and `lyx reed resume` the resumed process sees the same `LYX_STRAND_NAME` and `LYX_PARENT`;
`reed remove --name worker-2` and `--name <full name>` each remove the right strand;
a strand added in the prime is `<code>:<role>` and its process sees no `LYX_PARENT`.
`standalonegeom/naming_integration_test.go` (`integration` tag, real tmux) carries the standalone acceptance criterion at the engine `lyx webster run --target-dir` and `lyx burler run --target-dir` build, since a CLI-level run would spawn live Claude producers.
On a plain temp git checkout with no `.lyx-code`, it builds the reed engine exactly as the standalone wiring does, from `standalonestate.Derive` and `standalonegeom.ReedGeometry`.
It then adds a `worker` strand running `sh`.
The test asserts the state name and pane title are `<agentname.StandaloneCode(hash8)>:worker`, the process sees `LYX_STRAND_NAME` with that value, and `LYX_PARENT` is unset.

**ImpactSummary:** Adds two tmux commands to every strand launch and a title column to the pane listing every reed reader shares; no lookup changes.

**Commit:** 10: reed: mirror the full name into the pane title
