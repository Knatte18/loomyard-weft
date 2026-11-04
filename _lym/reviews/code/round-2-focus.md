---
round: 2
focus:
  - "Child process starts: PATTERN-reap-children requires a child's starter to wait on it by cmd.Wait or proc.Reap. Check internal/reedengine/lifecycle.go spawnSession (tmux server start), internal/vscode/launch_linux.go Launch and internal/vscode/launch_windows.go, and any other cmd.Start or os.StartProcess in production code that the diff touches or newly makes reachable from a long-lived process. Does each wait, or is the entry's text narrowed to name an exemption?"
  - "Startup reap coverage: internal/beamcli/cli.go runDobbyWith calls a reap seam after placeDobby, and internal/orchcli/watch.go calls the reap in the watch RunE. Does a test fail if either call is deleted or moved after a child start, and is the reap skipped when placeDobby refuses?"
  - "Zombie leader records in internal/runproc: record.go list and end.go end treat a zombie leader differently. Does list delete a record that end still needs to signal the group, and does a test with a real exited-unreaped child pin the production proc.IsRunning and proc.IsAlive choices?"
---

Focus restates the sites; it caps nothing.
