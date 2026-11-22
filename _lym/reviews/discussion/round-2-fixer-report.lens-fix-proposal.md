```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F8–F11) through the fix-proposal lens.

- **MEDIUM — F11 proposal, D4 step 2: "a beam qualifies only when its worktree's `_lym/landing/beam-ready.json` is absent".**
  The check reads a local worktree path, but the new parent can be another orch's worktree. D4 step 5 supports that case ("When the new parent is another orch's worktree"), and that worktree can sit on another machine, since the owner follows the parent.
  The ready record is committed on the beam branch (`internal/beamcli/cli.go:918-920`, "writes a ready record … and commits it"). So the read that works from any machine is the beam branch on origin, for example `git show origin/<beam branch>:_lym/landing/beam-ready.json`. A local path read finds nothing and passes a ready beam.
- **MEDIUM — F10 proposal, D4 step 3: "no records branch of the slug through `git -C <records repo> ls-remote origin`".**
  The text does not name the branch to look for. The records branch is the task branch name plus the weft suffix (`internal/fabricengine/branchname.go:11`).
  In a one-repository hub, records branches sit under origin's `refs/lym/heads/` (`pattern/PATTERN-single-repo-hub.md:3,17`). A reader matching `refs/heads/<slug>` finds nothing there and reads "absent" while the branch is live, which is the case the check exists to catch. The full ref name to match needs stating.
- **LOW — F9 proposal, D4 step 3 refusal list naming the owned-elsewhere refusal, plus D7 "lifts its pause with `lym tender resume <slug>`".**
  `lym tender resume <slug>` refuses a run another machine owns (`internal/tendercli/cli.go:200`), and the same ownership rule refuses the step-1 `lym tender pause <slug>` (`cli.go:164`).
  The owned-elsewhere refusal can therefore only come after ownership changed between the pause and the cancel. In that case D7's lift is itself refused, and no way forward is stated.

Out of lens: the fixer's own addition ("when the records branch is still listed after that, it stops … the entry left a draft under its old parent") stops before step 4. The move note then lives only in the orch's `.scratch/`, which D2 calls a copy that lasts only until the board write.

Tool calls:
- One grep of `internal/fabricengine/branchname.go` and `internal/tendercli/*.go`. My context lacked the records branch naming and tender's ownership refusals.
