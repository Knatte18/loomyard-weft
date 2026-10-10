# Fixing webster round 3's publish failure (reedcli leftover `sleep 300`)

The review put the leak in tmuxkit's sweep and proposed reaping pane trees there.
The kit's leftover line named only pid and argv, so tracing the process to its test took pid arithmetic against the server pids in the log.
That showed the server had been killed by reed's own `down` in `TestSmokeStateRecovery`, so the fix moved into reed's pane reap.
The leftover line now carries the process's cwd, which names the fixture hub directly.

Separately, `TestSmokeTeardown/DownForceKillsSighupImmunePaneChildren` turned out to be vacuous: `down` ran before the pane shell started its payload.
Any reed step that adds a strand and asserts on its processes needs to wait for the payload to start first.
