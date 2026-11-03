```yaml
lens: fix-claims
outcome: reported
```

fix-claims: I checked each claim the fix added or rewrote against the code at HEAD. One does not hold.

**List of claims checked:**
- D1 step 3, the fork-point checks the adopt-only option skips (the HEAD read and its detached-or-unborn refusal, the empty own marker refusal, the anchor-move stop): hold (`add.go:255-266`, `351-361`).
- D1, `Add` refuses a non-empty `Anchor` over a live weft: holds (`add.go:397-401`).
- D1, the stray path refused by `taskWorktreePresent`: holds (`wire.go:268-280`).
- D2, the checked-out refusal's own way forward `lym fabric checkout &lt;other branch&gt;`: holds (`addadopt.go:27`).
- D2, the stray-path refusal at the task worktree's location: holds (`add.go:298-307`).
- D6, "an archive tag of D4's covers that tip, at it or at a descendant": holds.
- D8 Rationale, `UpstreamSHA` returns a resolve error and `checkOrigin` passes it up: holds (`blobread.go:286-289`, `pushclassified.go:392-397`).
- D8, "An adopted local weft branch's origin copy is read by the weft probe": holds. `probeWeftLeftover` runs `remoteHeadTip` whether or not the branch exists locally (`remoteleftover.go:84-90`).
- Support log, `originTargets` reads tracking refs before the ls-remote: holds (`originreconcile.go:706`, `768`).
- Support log, "the ref was deleted after 08:18:46": holds. The pair push is a plain `git -c push.autoSetupRemote=true push` (`push.go:141`), which writes the tracking ref, so the 08:18:46 push would have left a reflog entry.
- D2 Land, the not-live bullet's filter "under the run id the batten status records": does not hold. See the finding.

**Finding:**

- **MEDIUM.** Changed text: D2, Land, the not-live bullet: "keeps a candidate whose commit holds this run's child status at `done` … under the run id the batten status records".
  - Summary: the shed status file records no run id, so the filter that tells this run's tag from an earlier run's cannot be applied.
  - Evidence: the recorded fields of `shedengine.Status` in `internal/shedengine/status.go:134-162` are `current_producer`, `state`, `error`, `transient`, `parent_notice`, `pending_by`, `entered_row` and the pause fields, with no run id. `RunID` in `internal/shedengine/shed.go:46-50` is a told address, not a recorded value, and the record's own Constraints say "Branch names carry no run id". The latest-committer-date pick still orders the candidates, but the claim that an earlier run's tag is filtered out by run id has nothing to rest on.

Calls made:
1. `sed` of `remoteleftover.go:60-160`: the weft probe's handling of an existing local branch, which my context did not show.
2. grep of the git push invocations in `push.go` and `pushclassified.go`: to test the reflog inference, since my context lacked how the pair push calls git.
3. `sed` of `shed.go:35-60` and grep for `RunID` in the status files: to test the run-id claim, since my context lacked the status struct.
4. grep of the `Status` struct's JSON fields: to confirm whether a run id is recorded, since call 3 showed only a told `RunID`.
