```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD (fix-claims lens).

Claims checked and found to hold:
- Loom's hand-over passes `InsidePair: true` (`internal/loomcli/landingdeps.go:926`, `internal/loomcli/wiring.go:235`), and batten's push leaves it false.
- `pushClassified` writes deferred records (`internal/fabricengine/pushclassified.go:499`) and transport records (`:532`, `:627`).
- `EndSubject`'s pass over the subject spares `RolePush` (`internal/runproc/end.go:100`).
- `HandOver` ends processes under the run's slug only (`internal/pairteardown/handover.go:103`).
- The order `skipsCheck` reads (prime, then pause, then Finalize) matches the pause-before-Finalize wording in D7.

**[MEDIUM]** D6, the prime-pusher bullet: "That run's unpushed prime commits go under an owner archive tag, and the replay carries only the other runs' commits."
- **Problem:** D6 states this as behaviour, but no code does either step, and D6 names only the `runSlug != ""` gate as the site that changes.
- **Evidence for the archive step:** `HandOverTips` archives "the pair's local code and records tips that origin lacks" (`internal/fabricengine/handover.go:58`). For a batten run that means the task pair's tips, not the prime's records commits.
- **Evidence for the replay step:** `retryWeftPush` replays every local commit onto origin's tip, and it does not filter by run (`internal/fabricengine/pushclassified.go:810` onward).
- **Missing from the record:** the change in fabric that archives the prime commits of a taken-over run, and the change that leaves them out of the replay.

Tool calls:
1. grep of `InsidePair` callers, of the owed-class record sites in `pushclassified.go`, and of `runproc.End`/`EndSubject` callers. My context had not yet shown who passes `InsidePair`, whether `pushClassified` writes deferred records, or which paths end processes by owner.
