---
format: 5
approved: false
language: go
---

# Plan: Keep a beam's pawl checklist in a window of its own beside the beam overview

While a beam's pawl run lives, `lym beam start`'s runs strand takes the session's overview window from the loom status strand, and the checklist is gone.
This plan builds the status strand's spec once, naming its own window for the run's recipe, and has `ensureRunsStrand` add the checklist back in that window in the same call.
Both adds go through reed's `AddStrandUnless`, so neither side of a race fails.
The beam overview gains one line with the worktree's own run and the checklist's window.

## Card Index

1 — status-strand-spec-builder — one builder of the loom status strand's spec and one sidecar path, in `internal/loomengine`
2 — loom-start-shared-builder — `lym loom start` builds its status strand through the builder and adds it through `AddStrandUnless`
3 — runs-strand-moves-checklist — `ensureRunsStrand` takes the overview window and adds the checklist back in its own window
4 — overview-self-run-line — `lym batten overview` shows the worktree's own loom run and names the checklist's window

## Shared Decisions

- The builder lives in `internal/loomengine`, which both `internal/loomcli` and `internal/orchcli` import and which already holds `LoomStatusStrandName` and `LoomScratchDir`.
  `internal/statuspane` was the record's other candidate, but it would gain `reedengine`, `shedrun` and `lymcwd` against the import set its `doc.go` states; `internal/loomengine` already reaches `reedengine` through `shuttleengine`, so it gains no cycle.
- Finished means a status whose state is `shedengine.StateDone`; an absent or unreadable status counts as no unfinished run, for D1's re-add and D3's line alike.
- Both callers read the self run's status through `statuspane.StatusCache`, the reader the overview already polls with, so no second status decoder is written.
