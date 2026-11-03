---
round: 1
focus:
  - "Every former `WorktreeRoot` use in `internal/websterengine` and `internal/webstercli` (e.g. `suspect.go`, `runwrites.go`, `recordbatch.go`, `recoverbatch.go`, `runlevel.go`, `beginbatch.go`, `reset.go`, `merriambase.go`, `verify.go`): was each moved to the root it means, worktree root for git and plan paths, `PaneCwd` for Webster's own `_lym`/`.lym` state? Look for a use left on the wrong root, since the record requires each use checked."
  - "Removal of `RepoRoot` in `hubgeom` and `standalonegeom` `webstergeom.go`: do both geometries still agree, and does any caller outside the diff or any standalone (non-hub) case now resolve a different path than before?"
  - "Does any Webster code join or strip the anchor by hand instead of reading `lymcwd.Location`, against `PATTERN-cwd-resolution`?"
  - "Do the begin-batch test and the plan-path tests (`path-missing`, `planglyph.Resolve`) run in a hub with a non-`.` anchor and a `root:` plan, so both #32 and #31 fail without the fix? Check the `verbs_test.go` fixture change, which drops the anchor `.git` link."
  - "Test economy: do the many one-line edits in `*_test.go` helpers add assertions or tests that duplicate existing coverage, and is the new behavior covered once?"
  - "Does the commit message's claim that `RepoRoot` is now always equal to `WorktreeRoot` hold in the diff, and are `PaneCwd` consumers (transcript-relative write paths, fork audit, gate cwds, prompt rendering, instruction files) all switched?"
---

Seed pass: focus drawn from the record's Goal and the diff stat of `fe4cd15e..HEAD`.
