# Support log — tender-pause-holds-reprompt

## Interview

### Turn 1 — exploration (autonomous; no operator)

Read the board entry, `run-process-ownership` and `batten-cancel-verb`, then the pause record, tender, creel interrupt, shuttle's gate loop and send path, the three verify sites and webster's batch verbs.
Findings that shaped the design:

- Shuttle's gate loop re-prompts through `sendWithin` → `sendVerified` with no pause check; only run start (`waitWhileCovered`), the deadline and the held-turn notices read coverage.
- `verifytree.Verify` already kills its command's process group on context cancel, but every gate caller passes `context.Background()`.
- Creel refuses an interrupt under a pause because an unrecorded interrupted session would get no resume line; tender re-pause works around it by recording the session.
- `pauserecord.CoveredError` names `lyx resume` as the way forward, which Master must never run.

### Turn 2 — design picks (auto)

Settled D1–D8 as recorded; see the question ledger.
The incident's darn session was `already_idle` at the pause, so it was never recorded; D3's hold keeps its re-prompt back, and D1 is the backstop.

### Turn 3 — lens review

Every fork returned a report.
Duplicates merged across lenses:

- interactions F1, walkthrough 1, bound-breaker 2, existing-paths 4 (D1 reads the runner's slug, and tender, creel and orch runners have no geometry): fixed; D1 keys on the target strand's slug and every production runner is told the hub geometry.
- goal-scope 1, bound-breaker 1, existing-paths 1 (D7 turning `Paused` into the claim would claim every strand with queued mail): fixed; the claim is a separate seam method used only by interrupt.
- interactions F2, bound-breaker 3, existing-paths 3 (creel delivery is an unlisted chokepoint caller): fixed; D2 maps `ErrPaused` to a held outcome.
- walkthrough 2, bound-breaker 5, existing-paths 2 (SIGKILL on the group skips `lyx gate test`'s handler and leaves test binaries running): fixed; D4 sends SIGTERM first, SIGKILL after the wait delay.
- walkthrough 3, bound-breaker 6 (D8's Master GUID unstated): fixed; resolved through reed's `ResolveStrandGUID` on `LYX_STRAND_NAME`, failure warns and still refuses.
- bound-breaker 4 (resume between claim and key): fixed; D7 re-reads the entry under the pane lock before the key.
- existing-paths 5 (loom status reads `paused` marker as running): fixed; D4 says status shows paused.
- existing-paths 6 (notice retry without backoff): fixed; D2 doubles the interval up to five seconds.
- interactions F3, bound-breaker 7 (`Expired` probes under cover): fixed by stating it in D3 as existing behaviour accepted.
- existing-paths 7, walkthrough 5 (finalize blocked by a held Verify): fixed; added to Open risks.
- interactions F4 (orch watcher on an already-claimed orch): fixed; checked `orchengine/pause.go`, which marks the pause handled with no Esc, and D6 says so.
- walkthrough 4 (non-orch prime strand under D7): rejected; tender resume sends its line to every recorded non-orch session by GUID, prime ones included.
- goal-scope 2 and 3 (recover-batch, D2/D6 past the entry's letter): rejected; their rationale is stated and they follow from the entry's "no new one starts" and from D1.
- goal-scope 4 (Windows as limitation): fixed; moved to Open risks.
- leanness (Scope In restates decisions, Goal list, API-continue bullet, duplicate constraint, notes, auto-mode restatements, Open risk restating today's behaviour, missing `(test)`): fixed, each cut or shortened; the incident note moved here.
- board-fit 1 and 2 (held Verify vs `run-process-ownership` and `batten-cancel-verb`): fixed; a plan-writer note says the hold ends with its process or context.
- board-fit 3 (D8 vs `pause-background-work`): fixed; a note says D8 settles only the no-new-batch half.
- board-fit 4 and 5 (`creel-agents`, async verify notes): rejected; no conflict today and nothing unbuilt to align.

## Rejected alternatives

- Pause check in each caller of shuttle's send instead of the chokepoint: misses future callers and the CLI paths.
- `sendVerified` waiting while covered instead of refusing: blocks CLI verbs indefinitely.
- Changing `shuttleengine.Gate` to take a context the runner cancels on a pause: touches every gate closure for one consumer, and leaves landingshed's verify outside shuttle unheld.
- Letting a running verify finish under the pause and holding only its result: the incident's verify looped for many minutes, and a hibernate needs the processes quiet.
- Stopping verify at the instant of the pause: discards a nearly finished run the pause's own grace would have let finish.
- Keeping creel's refusal and documenting the re-pause workaround: the board entry asks for the interrupt to work.
- Reusing `CoveredError` for the webster refusal: its way forward is `lyx resume`, which lifts the operator's pause.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): where does the "no prompt under a pause" rule live? → shuttle's `sendVerified` chokepoint plus a gate-loop hold (D1, D3).
- Q2 (auto-pick): stop or finish a running verify? → stop after the entry's grace end, hold, run again (D4).
- Q3 (auto-pick): which verify sites? → all production `verifytree.Verify` sites, parent-lock holders hub-only (D5).
- Q4 (auto-pick): how does creel interrupt stay safe for the resume? → claim into the covering entry first (D6, D7).
- Q5 (auto-pick): include `recover-batch`? → yes, it forks too (D8).
- Q6 (auto-pick): what does a refused tender resume line do? → claim into the covering entry (D2).
