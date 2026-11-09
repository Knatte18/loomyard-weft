# Relevant files

- `internal/adoptcli/doc.go` — the full adopt step spec, holds, deploy, owner write, beam steps, check, green end and run report
- `internal/dobbyengine/loop.go` `daemon.loop`, `daemon.beamOwnedElsewhere` — the dobby's exit on a foreign owner, where D4 and D11 hook in
- `internal/orchengine/owner.go` `Watcher.ownerTick`, `OwnerWatch` — the orch watcher's owner stop that D4 suppresses
- `internal/fabricengine/archive.go` `ArchiveTagName`, `archiveTaskTip`, `archiveWeftTip`, `tagAndPushArchiveTip` — the archive tag scheme and push D11 reuses
- `internal/shortcutcli/shortcuts.yaml` — the `s` shortcut D10 changes
- `internal/shortcutcli/shortcut.go` `runArgvs` — stops at the first non-zero exit, which D10 relies on
- `internal/pauserecord/record.go` `OwnerCaller`, `HoldCaller`, `Update` — pause callers and the record update D5 extends
- `internal/orchcli/start.go` `BeamUp`, `BeamUpOptions`, `BeamUpWorktree` — the beam steps D7 gives an orch skip
- `internal/fabricengine/junctionnames.go` `AdoptScratchDir` — where the adopt record sits
- `internal/runproc/record.go` `HubOwner` — the owner the worker registers under
- `internal/fabricengine/bolt.go` `Bolt.Pull` — the Bolt pull D10's `--here` and D11's periodic owner read use
- `internal/fabricengine/hubpull.go` `PullHub` — the hub pull, reached only from `lym fabric pull`, so no periodic pull exists today
- `pattern/PATTERN-dobby-runs.md` — the adopt-hold and dobby-orch rules D5 and D7 change
- `pattern/PATTERN-owner-machine.md` — owner rules the owner write order and the archive push stay within
- `pattern/PATTERN-child-processes.md` — spawn, reap and runproc rules for the worker
