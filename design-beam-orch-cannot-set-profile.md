# Beam orch cannot set a board entry's profile: refused as a task session

`beam-orch-cannot-set-profile` · Task · bug, board, orch, workspaces · running · Record-Write

Imported from [issue #12](https://github.com/Knatte18/loomyard/issues/12).

Seen: in a beam's orch (strand mo:loomyard-setup:orch), `lym board upsert` with `"profile":"docs"` on a new task under the beam's project was refused: "a run_settings or profile change is refused from the task session mo:loomyard-setup:orch; way forward: the operator or the hub orch makes the change". The same upsert without `profile` succeeded, so the beam orch may create tasks but not set their profile.

Expected: the orch role tells every orch, a beam's included, to name a profile in the board entry's `profile` field before the batten run starts. A beam's orch (role `orch` under a beam slug) should be treated as an orch, not as a task session, for profile and run_settings writes on entries whose parent_beam is its own beam.

Workaround: created the task without a profile and asked the operator or the prime orch to set it.