```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each review proposal the fixer carried out as proposed, from the fix-proposal lens.

Findings:

1. MEDIUM — D7 relaunch steps 1–4 (F1 proposal)
   - Changed text: "Shuttle's stop verb, in a new relaunch mode, ends the limited session's process and removes its reed strand… 4. The new strand starts in the same run."
   - Summary: the proposal never says the run record takes on the new strand's identity, and Wait's own liveness check would then end the relaunched run as died.
   - Evidence: `checkLivenessTick` looks the strand up by `run.state.StrandGUID` (`internal/shuttleengine/wait.go:1462`) and returns `OutcomeDied` when reed no longer tracks it (`wait.go:1468`, `wait.go:1484`). The record has two possible fixes and names neither: reuse the strand GUID, or rewrite `StrandGUID` in the run record when step 4 starts the new strand.
   - Evidence: between steps 1 and 4 the record says the run is live while its strand is gone. That is the state `PATTERN-shuttle-provider-seam`'s Shuttle stop section warns "an attach probe or a Wait then reads". Nothing covers a waiter that crashes, or a `Prepare` that fails, inside that gap.

2. MEDIUM — D7 "the API-error watch is reset when the relaunched strand starts" (F5 proposal)
   - Summary: "reset" can be read as a fresh `apiErrorWatch`. That watch would re-read the limited turn end and re-send it into the new strand, the opposite of what the proposal intends.
   - Evidence: the watch's cursor "starts at the prompt offset, so an attach counts the run's earlier API errors" (`apierrorretry.go:26`). The relaunched strand writes to the same run's `EventsPath`, so a fresh watch reads the handled `SessionSignalAPIErrorTurnEnd` again as pending (`apierrorretry.go:63-65`). The criterion "records no API-error re-send for the limited turn end" then fails.
   - The record should say the cursor keeps its position past the limited turn end, and only the count and the pending state clear.

3. MEDIUM — D6 fallback "an API error the engine classifies as a refused resume… the orch through its start's fresh launch" (F4 proposal)
   - Summary: no such classification exists, and D6 names no signal to build one from.
   - Evidence: D6's own Facts say foreign thinking is "dropped elsewhere with no error". The new acceptance criterion tests a classification whose trigger is undefined.
   - Evidence: the orch part has no path. By F6's fix the watcher is told only a resume-backed restart capability. A fresh start is `orchcli`'s `startSession` path (`internal/orchcli/start.go:784-793`), which the record gives the watcher no way to reach.

4. MEDIUM — D1 "a listed directory that does not resolve is refused at load" (F15 proposal)
   - Summary: one missing account directory now fails every session launch in the hub, including unpinned worktrees with no entry on `~/.claude`.
   - Evidence: D1 says "the session launch fails; a bad value never degrades to no accounts".
   - Case: a retired account's directory, kept listed at weight 0 for history (D1, D8), is deleted or sits on an unmounted disk. Every lym launch in the hub then refuses, though no session uses that account.
   - D3 step 1 already handles an account that is not logged in by a credentials stat. Comparing a missing directory cleaned, as D8 does for the non-account directories, would fix F15 without this outage.

5. LOW — D2 "A pick drops the entries whose worktree name is absent from the hub's current worktree names, which the caller … passes" (F7 proposal)
   - Summary: names read before the lock can drop a sibling's fresh entry.
   - Case: two worktrees X and Y are made at once. X's caller reads the hub's names before Y's directory exists. Y's pick writes Y's entry. X's pick then takes the lock and drops Y's entry as absent, so Y runs as today, unpicked.
   - The proposal does not say the names are read under the account file's lock, or that the newly made name is never pruned.

Tool calls:
- grep for `NewWatcher` and `checkLivenessTick` in `internal/orchengine`, `internal/orchcli` and `internal/shuttleengine/wait.go`: my context lacked where the orch watcher is built (for F6) and how Wait classifies a strand reed no longer tracks (for finding 1).
