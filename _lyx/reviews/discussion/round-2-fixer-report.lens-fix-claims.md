```yaml
lens: fix-claims
outcome: reported
```

I checked the claims the fixer added to the decision record against the code at HEAD. Most hold, and one is too narrow.

**MEDIUM: other owned-elsewhere refusals still loop as failing passes**
- **Changed text:** the D4 paragraph "A full pass refused as owned elsewhere (`resumeRefusal`'s `EntryOwnedElsewhere` arm) exits with a code of its own…", and the matching acceptance criterion.
- **Problem:** the new exit code covers only one of the owned-elsewhere refusals a full pass can return. The others still exit as plain errors, so dobby retries them as failing passes, which is the loop F1 was meant to end.
- **Evidence:**
  - `internal/battencli/owner.go:182-183`: `resumeOwnerClaim` returns `ownedElsewhereRefusal` when the owner write is lost (`write.Lost[slug]`).
  - `internal/battencli/owner.go:276-277`: with origin unreachable, `refuseResumeUnreachable` refuses with a plain `fmt.Errorf` when the local board shows another owner.
  - Both are reached from `reconcileResume` (`internal/battencli/resumereconcile.go:52-62`), so a full pass offline, or one that races an adopt, still loops.
- **Related claim:** "resumes on the local Bolt, as `refuseResumeUnreachable` does today" is only partly true. That function judges the owner on the local Bolt and may refuse, so the unreachable path has to say what happens on that refusal.

These claims hold:
- InnerRun's `check()` order: `coverage`, `foldOwnerTrigger`, `checkChild`, `ownerStep`, `refreshWatched` and `lockStep` on a due probe, `noticeStep`, the arm's step, then the pause-flag read (`internal/battenshed/innerrun.go:696-714`).
- `deployTimeout` is 30 minutes (`internal/battenshed/land.go:33`).
- The `lyx loom start --no-attach` bootstrap uses `exec.Command` with no context or timeout (`internal/battencli/wire.go:1205-1213`).
- `tendercli.PauseOwnedElsewhere` is InnerRun's owner-pause seam (`internal/battencli/wire.go:951`).
- `ownedElsewhere`'s parent-lock release is kept by `keepsLockForFinalize`, so "the same parent-lock release" is accurate.
- `lyx adopt` takes several slugs in one call (`internal/adoptcli/adopt.go:214-229`), so "unless the same call adopts the beam" can be carried out.
- "Three in-pass waits last as long as what they wait on": `holdPause`, Land's land-lock poll and the bootstrap.

Not verified: D5's claim that an upsert setting `status` to null clears it. Two greps in `internal/boardengine` did not find the patch code, so I left it unjudged. It rests on the reviewer's citation of `store.go:436-448`.

Tool calls:
- `grep` of `refuseResumeUnreachable`'s body and the board's `status` key: my context did not show what the unreachable-origin path does.
- `grep` for `ApplyPatch`'s status handling: my context lacked how a patch treats a null status.
- `grep` for a `case "status"` in `internal/boardengine`: same reason, after the first search found nothing.
- `grep` for `"status"]` or `Status = nil` in `internal/boardengine`: same reason, after the second search found nothing.
