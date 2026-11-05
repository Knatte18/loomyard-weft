---
round: 2
focus:
  - "internal/boardengine/runlock.go and internal/boardengine/sketch.go: settle whether a run-held task that becomes a status-less Sketch note while its title, brief, body, short name, project, beam, order, issues, priority, front, on_land or slug also changes is still refused by checkRunLocks with ErrRunLocked, and whether a held task that becomes a note without the sketch label is refused."
  - "internal/battencli/startguard.go, hubStartGuard's Unreleased closure: settle whether a task still carrying the legacy discussed status, which another entry depends on, is refused by `lym batten run`, and whether the guard's doc comment, the Unreleased field comment and the release command text name that status consistently with the code."
  - "internal/loomcli/wiring.go, sketchFinish: settle whether the fall back to draft happens only when the board refuses the Sketch write, whether any other MakeSketch error is returned for the seam to retry, and whether a test reaches the refused branch with an entry another entry lists in depends_on."
  - "internal/boardengine/sketch_test.go: settle whether the tests pin the exemption boundary through checkRunLocks or UpsertTask, whether the held-task conversion asserts the cleared fields (isolated, profile, pull_request, notify_on_land) and the kept ones (project, brief, body, issues, dates), and whether a depended-on discussed task, a replayed Sketch write on a held entry and MakeSketch on an existing Sketch are covered."
---

Each entry restates its sites so the reviewer needs no earlier round.
