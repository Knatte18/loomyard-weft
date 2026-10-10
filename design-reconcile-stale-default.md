# Reconcile keeps an old default below the new floor

`reconcile-stale-default` · Note · enhancement, config

Seen (2026-10-10): after `daemon-wakeups` raised `poll_interval_ms` in the shuttle template to 1000 with a floor of 1000, `lyx config reconcile` kept the prime's and every pair's old default `500`, so every `lyx` command in them logs "shuttle config: poll_interval_ms is below the floor and is raised to it".
The orch set the prime to 1000 by hand with `lyx config shuttle --set`; pairs created earlier keep warning until they are torn down.

Idea: reconcile updates a value that still equals the old template default, or at least one that the new floor would raise, and says so in its report.
