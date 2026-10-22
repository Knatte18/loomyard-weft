# Round 1 fixer-report — machine-handover discussion

Every finding's premise held against the code; each is fixed in `_lyx/discussion/decision-record.md`, with the round and its auto-picks recorded in `_lyx/discussion/support-log.md`.

## Changes

- F1 (BLOCKING), fixed as proposed: D7 bullet 3's message now depends on the failed check.
  A parent check names the parent's owner, adopting the parent here (`--prime` or `<parent-slug>`), or `lyx adopt <slug>` on the parent's owner.
  A run check names the run's owner and `lyx adopt <slug>` here.
  D5 step 5 resumes an owner-halted child when its own slug is adopted and when `--prime` or a parent slug adopts its parent while this machine owns its run, so the message carries no separate resume.
  The self-check added the both-fail case (see Self-check).
- F2 (MEDIUM), fixed differently: D5 step 1 stops a named owned_elsewhere slug whose branches the reconcile left in place, before its owner write, and the other slugs go on; a criterion covers adopting back.
  The proposed way forward, removing the stale pair through fabric and re-adding it from origin, was not made: `lyx fabric remove` also deletes the pair's branches on origin (`internal/fabricengine/remove.go`, `RemoveStepTaskBranchOnOrigin`, `RemoveStepSiblingBranchOnOrigin`), and it keeps a local warp branch with unlanded work, which then refuses the re-add (note `fabric-add-adopt-local`).
  The ways forward are new adopt flags instead: `--keep-local` keeps the commits under the new owner, and `--drop-local` archives each local tip under a local tag and moves the branch to origin's tip.
- F3 (MEDIUM), fixed differently in its mechanism: D6 releases the child's run parent lock on an owned_elsewhere verdict before the owner pause, and the released run takes the lock again at its next lock-taking row; D7 bullet 2 points at it; a criterion was added.
  The release goes through InnerRun's existing Release seam, which passes the record's own owner (`internal/battencli/parentlock.go:34`), instead of a "Release by its own owner process", which would not match a record the child's driver process owns (`internal/parentlock/doc.go`, Release and hand-off).
- F4 (MEDIUM), fixed as proposed: the release lets the slug entry hold the run (D6), D4's wake and orch-start rows say their pause holds a lock-holding run only after that release, and InnerRun also releases when it reads an owner-labelled pause over its slug.
  D6's Bound names the landing row past its lock take and the call past the network gate, and Open risks names a run with no live batten.
- F5 (MEDIUM), fixed as proposed, the refusal-as-verdict option: InnerRun's re-step spawn passes the check, and its owner-kind refusal is read as an owned_elsewhere verdict (D3, D6); D3 bullet 5 covers the shed verbs with origin unreachable.
- F6 (MEDIUM), fixed as proposed: batten's start pulls Bolt and reads the owner before any seed or claim, a slug with no board entry writes no owner, and the claim refuses the Worktree-Create row on a failed owner write or push, resuming with the same verb (D3).
- F7 (MEDIUM), fixed as proposed: `--all` covers only tasks that are not done; step 5 resumes only a task with a not-done run record, and a task with no run record gets its owner and no run.
- F8 (MEDIUM), fixed as proposed: the Constraints line states the rule as a class and lists the local-Bolt reads; D3 bullet 5 refuses a task the local Bolt shows as owned elsewhere while origin is unreachable.
- F9 (LOW), fixed as proposed: D3 says batten's resume, the shed resume verbs proceed on an unowned run-held task and write no owner, and Open risks says next batten resume.
- F10 (LOW), fixed as proposed: D4 gives owner pauses one fixed `Caller` label and adds no entry over a standing one; D5 step 4 lifts by scope, caller and paused time and names a standing pause's lift.
- F11 (LOW), fixed as proposed: D3 re-reads the owner after a start's or a resume's owner-write push and refuses on a lost write.
- F12 (LOW), fixed as proposed: criteria were added for each listed behaviour.
- F13 (LOW), fixed as proposed, the stated-as-intended option: D8 covers a same-machine resume with no client attached, and the D8 criterion includes it.
- F14 (NIT), fixed as proposed: the rewording clause was deleted and its behavioural remainder kept as its own sentence.

## Self-check

- fix-claims, MEDIUM, revive does not spawn `lyx loom start`: fixed as proposed. D3 names only the re-step spawn and says bootstrap and revive meet no shed verb, with D6's checks bounding both.
- fix-claims, MEDIUM, the shed verbs have no unreachable-origin Warn today: fixed as proposed. D3 says they run no reconcile today and gain the same Warn and resume.
- fix-claims, LOW, a halted uncovered child does not keep its lock: fixed as proposed. D7 bullet 2 follows batten's lock step as it is, and D6 names its release as the one release under a pause.
- fix-consistency, fix-proposal and fix-way-forward, MEDIUM, merged: the both-fail message offered adopting the parent here, which resumes nothing. Fixed: it names `lyx adopt <slug>` on the parent's owner, or adopting both here in one call; D5's Arguments allow `--prime` with slugs; a criterion was added.
- fix-consistency, LOW, D5 step 3 still recorded every named entry: fixed as proposed, so only slugs that steps 1 and 2 did not stop are recorded.
- fix-consistency, LOW, step 1's stop was not limited to owned_elsewhere slugs: fixed as proposed.
- fix-consistency and fix-proposal, LOW, merged: D6 omitted the spawn refusal as a verdict source, and the bootstrap disagreed with "on an existing run". Fixed: D6 names the re-step refusal, and D3 excludes the bootstrap.
- fix-consistency, LOW, Scope In lacked the new flag: fixed as proposed.
- fix-consistency and fix-proposal, LOW, merged: Turn 4 points at the sentence F14 deleted. Fixed: the support log's Round 1 entry records the removal, and Turn 4 stays as history.
- fix-consistency, LOW, a non-caller row in D4's caller list: fixed as proposed by folding it into the wake and orch-start rows.
- fix-proposal, MEDIUM, a released lock strands Finalize's unpushed local parent merge: fixed. Such a run keeps its lock, and its notice names `lyx adopt <slug>` here (D6), with a criterion exception.
- fix-proposal, LOW, an unlabelled standing pause gets no owner pause, so no label trigger fires: fixed differently, by stating the window in Open risks, since a new trigger would run a reconcile on every pause of any origin.
- fix-proposal, LOW, D8 reopens windows on orch resumes after a deliberate detach: fixed. The overstated rationale was cut, and D8 names turning `open_window` off through run settings.
- fix-way-forward, MEDIUM, `lyx resume <slug>` is refused for a hub-only cover: fixed as proposed. D5 step 4 names `lyx resume` for a hub entry.
- fix-way-forward, MEDIUM, the branch repair steps push the old commits before the takeover: fixed. That way forward was replaced by `--drop-local`, and neither way forward pushes before the owner write.
- fix-way-forward, LOW, the release is reached only after a sleep or a pause event: fixed by the D7 bullet 2 change above. An uncovered halted child's lock is already released by the idle bound, and D6 covers the paused case.
- fix-way-forward, LOW, a start refused after its push leaves a stopped run record that adopt would take over: fixed differently, by naming it in Open risks with the race it belongs to, since it arises only when two machines start the same task at once, which the premise excludes.

## Disputed

None.

## Deferred

None.
