```yaml
lens: fix-claims
outcome: reported
```

Fix-claims lens: checked every bound, guarantee and code fact the round-1 fix added to `decision-record.md` and `support-log.md` against HEAD.

Claims checked that hold:
- `.weft/` is git-excluded (`recordsgit.go` comment on `seedWeftArtifactExcludes`).
- `EndSparing` exists and spares roles (`end.go:60`).
- `RolePush` and `RoleSync` exist (`record.go`).
- A dobby start owes every run a full pass, and a held driver lock reads as in flight (`dobbyengine/doc.go`).
- `EnsureLive` is called from `lyx orch start`, `lyx adopt` and `lyx beam start`.
- Reconcile's wiring repair wires at the marker's anchor and moves no content or launchers.
- `settleUnmovedWeft` replays local commits onto origin's (`pull.go:325-327`).
- The weft push replays on a records rejection (`pushclassified.go:2,88`).
- `lyx tender pause [<slug>]` takes a slug.
- `lyxcwd` has no notion of the prime.

Findings:

1. MEDIUM. D7 step 2, "the records pushes and board syncs registered under `runproc.HubOwner` are spared", and the support-log line "pushes are spared through `EndSparing`".
   Summary: the spare does not reach a push that sits below an ended dobby or pass in its process tree.
   Evidence: `end.go:115` takes a spared record out of `todo`. But the descendant walk (`end.go:316-345`) skips a child only when it is `Zombie`, `claimed` or `groups[child.PGID]`, and never calls `skip`. A spared record is never `claimed`, so a push still parented by the dobby, or by a pass spawned in the window step 2 admits, is signalled. That is exactly the window step 2 says it ends "as a crash ends one".
   The guarantee holds only for pushes and syncs outside the ended dobbies' process trees. The text should say so, or name what carries a push ended that way.

2. LOW. D7 step 8, "start the worktree's dobby when it is a beam and its lock reads free, as `dobbyengine.EnsureLive` starts one".
   Summary: `EnsureLive` also leaves out a beam that another machine owns, and step 8 does not.
   Evidence: `start.go:81`, "a beam another machine owns is left out of the report". The fix lets a non-owner machine run an integrating `--set`. Step 8 would then start a dobby there that exits at once, since the loop exits when another machine owns the beam (`doc.go`). The claim "as `EnsureLive` starts one" is therefore not exact.

Outside this lens, for the way-forward fork: `lyx tender pause <child>` does not stop continue passes, because "While a pause covers the run, only continue passes run". The refusal on a child pass in flight can therefore keep firing after a pause.

Tool calls:
- `pull.go:320-345`, tendercli `Use:` lines, pushclassified replay grep: needed to check the claims about `settleUnmovedWeft`, the tender pause slug and push replay, which my context lacked.
- `end.go` spare/skip grep: needed to check the claim that `EndSparing` spares pushes, since my context did not hold the spare logic.
