# Merge the remaining adopt-build members into one design and split it again on what landed

`adopt-build-resplit` · Task · feature, enhancement, batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut · running · Discussion-Bouncer

## Why (operator, 2026-11-16)

Raddle split the Sketch `adopt-redesign` into the members of the beam `adopt-build` before any of them ran.
Two members have landed in the beam since, `orch-resume-own-session` and `run-fsm-tracked-records`,
and the operator has decided more since the split:
the orch resumes its own session on the same machine only, with no transcript mirror;
the orch stays up during an adopt hold;
the stop button, the restart launcher and the root-beam fix joined the beam.
The remaining members were cut for a state that no longer stands.
This entry merges every remaining member into one design, to be split again from scratch under raddle's split rules,
on what landed and what the operator decided.
Splitting, merging or reordering the parts is open, and so is moving a part out of the beam.

## Input

- The beam's branch as it stands, with the two landed members' code, such as `internal/runfsm`.
- The Sketch's decision record, under Bolt's `raddle/adopt-redesign`.
- The study `adopt-build-revision`, `docs/research/loom/adopt-build-revision-2026-11-16/report.md` on the beam's branch.
  Use its findings on what the landed members built and what each remaining part takes from them.
  Its proposed shape, each member kept and one added, was rejected by the operator: never take it as the split.
- The merged members' designs, below, as each stood when merged.

## Merged members

### push-before-act-takeover

Every transition pushes its records before the run acts, and a takeover entry in the run's records hands it over without replay.

Takes: runfsm's Act inputs and the batten record from run-fsm-tracked-records; fabricengine's PushPairAnchored, SpawnStatusPush, teardown-lock respawn, pushclassified's rejection path and ArchiveOwnerTips; boardengine's Sync, SpawnSync and pushOwnerCommit; statuscommit and sheddriver's records commit; landingshed's Finalize; runproc's End for a slug

Delivers: a shed transition that takes a gate admission, commits its records with the code tip it was made with and pushes records side first, code side after, before the next producer starts; an owed push retried on a backoff and shown in lym beam status, which tender's offline pause and resume intents also ride; the per-transition detached push and its teardown-lock respawn retired; synchronous Bolt writes for a run claim, a landing's done and an owner; Finalize pushing the parent branch before its transition and board done; a rejected records push that reads origin's record and hands over on another machine's takeover entry, archiving local tips and the local parent merge; the code-tip wait and its grace

Exports: the takeover entry type and its reader, tested with kit-written entries; the owed-push and code-tip facts runfsm's Act reads; fabricengine's records-first PushPairAnchored and its hand-over result

Packages, by the Sketch's estimate: fabricengine (about 1000 changed lines); battencli, sheddriver, loomcli, landingshed (about 200 each); boardengine, statuscommit (100 to 150); shedengine, beamcli, fabriccli, tendercli, tenderengine, pauserecord, pattern (under 60 each).

### bolt-holds-catch-up

A machine registry and holds in Bolt, the owner pause caller retired, and a catch-up from origin after an absence.

Takes: runfsm's Act rule, the synchronous Bolt writes, the takeover reader and hand-over and the owed-push facts from push-before-act-takeover; the subject-process end from orch-resume-own-session; boardengine's owner setter and OwnedElsewhere, machinename's former names, pauserecord's callers and SleepWatch, the dobby's loop and handover, battencli's origin-owed marker, pairteardown's local mode

Delivers: board.json's machines registry and holds list written only by boardengine's setter with a synchronous push; every machine-name comparison through the registry, OwnedElsewhere's callers in every package updated; lym machine name --forget-former; hold slug entries under the new pauserecord caller hold, read by every machine; the owner caller and its writers retired into hand_over with the subject-process end; the origin-owed marker retired; the catch-up from origin after a sleep, a dobby start or a hand pass, pushing owed commits, handing over runs owned elsewhere and removing their local pairs in local mode, the dobby's handover calling the subject-process end

Exports: boardengine's registry and hold types, setters and readers, OwnedElsewhere taking the registry, and the catch-up entry point adopt calls as its step 4

Packages, by the Sketch's estimate: boardengine (about 600 changed lines), dobbyengine (about 450); pauserecord, machinecli, orchcli, tenderengine, battencli, beamcli (100 to 150 each); tendercli, hibernateengine, adoptcli, machinename, loomcli, battenshed and a tail of small ones (under 60 each).

### adopt-foreground-leftovers

lym adopt runs in the foreground over pushed state, with --take-hold, and the dobby tears down or reports leftover pairs.

Takes: the holds, registry, catch-up and owner setter from bolt-holds-catch-up; the takeover entry type, its reader and the synchronous pushes from push-before-act-takeover; runfsm's states; adoptcli's fix loop, check, deploy, archives and here paths; the dobby's ended-run teardown and pairteardown

Delivers: lym adopt running hold, owner, pull, takeover, catch-up, deploy, workspace, check and green clear in the caller's process, re-runnable from tracked state, with the takeover entry writer and its rejected-push retry, and --take-hold denied to agents by a shell-guard row; the worker, adopt record and its readers, verdicts, pause step, the adopt and adopt-hold pause callers, --keep-local, --drop-local and --discard retired, with the upgrade's first read turning standing adopt entries into pause intents and one Bolt hold; the dobby tearing down landed and unkept cancelled pairs, listing kept pairs, writing kept-cancel records for pre-upgrade kept pairs, and tearing down idle orphans after the orphan grace; lym tender resume pushing no pair; the hub workspace file regenerated after each teardown

Exports: adopt's foreground step runner, the takeover entry writer and the orphan grace constant

Packages, by the Sketch's estimate: adoptcli (about 2650 changed lines), adoptrecord (about 400), dobbyengine (about 350); fabricengine, beamcli (about 150 each); orchcli, pauserecord, battencli, shellguard, tendercli, pairteardown, cmd/lym, pattern (under 80 each).

### prime-is-root-beam

The prime is the root beam: parent_beam names it, never empty.

Seen (operator, 2026-11-15): `lym board list` and `lym board get` give `parent_beam` empty for an entry whose nearest beam is the prime, and `lym board list --text` shows no `P0`,
while the board view and window titles show the prime as `P0`.
The prime is a beam, the root one; an empty value reads as "no beam".

- `parent_beam` names the root beam, as `P0` or its slug, for every entry under the prime, never empty.
- `lym board list --text` shows `P0` with the other projects, so every view of the board shows the same tree.
- Every reader of an empty `parent_beam` (orch role, dobby, board verbs) moves to the named root in the same change.

### hub-stop-button

A stop button in the prime orch session that takes the whole hub down, behind a yes/no. Imported from [issue #122](https://github.com/Knatte18/loomyard/issues/122).

Seen (operator, 2026-11-11): to stop everything on a machine without a reboot, the operator runs several verbs by hand:
`lym beam stop` for the orch and dobby of the prime and of each beam, `lym reed down` in each worktree for the tmux sessions, and `lym tender reap` for run processes no run owns.
It should be one simple action.

- A button in the reed status bar, shown only in the prime orch's session, that takes the whole hub down, after a yes/no confirmation.
- It ends the orch and dobby of the prime and of every beam, the reed sessions of the runs' worktrees, and run processes no run owns.
- Whether it first pauses the hub, so a run's turn is interrupted cleanly, and what the confirmation says, are open.
- The operator closes terminal windows (Konsole, the VS Code terminal) by hand.
- Built on the run FSM and its pause and cancel intents (operator, 2026-11-15), so a stop leaves every run in a pushed state another start or another machine goes on from.
  The adopt decision record kept a hub-wide pause machine-local; this button needs no shared pause between machines either.

### machine-restart-shortcut

One desktop launcher brings back every hub and beam that was in flight after a crash or reboot.

Seen (operator, 2026-11-08 and 2026-11-11): OSL-1046 and Neve crash often.
After a crash the operator finds every hub and project that was in flight, often three or four with beams under them, and starts each orch session by hand.
Once an orch is up, it and its dobby take the runs up again by themselves; a run whose step was cut off halts (`loop-exited`) and the orch settles it.
Finding what was running is the hard part, not resuming it.
VS Code reopens its own windows on Kubuntu.

- A `lym` verb starts again every project that was in flight on the machine: the hub prime and every live beam under it.
  It opens the hub's VS Code workspace file, `<hub>/<repo>.code-workspace`,
  and for the prime and each beam a terminal outside VS Code in the worktree running `lym s`, which starts its dobby and orch and attaches to its reed session.
  It opens no task window and resumes no run; the orchs and dobbies take the runs up.
- Desktop launchers call the verb: a `.desktop` file on Linux, a `.cmd` file on Windows.
  lym adds and removes them as hubs and beams come and go, and removes a hub's launcher when another machine takes the hub over.
- It starts only what this machine owns: it reads the owner of the hub's prime and of each beam, the owner `lym adopt` writes,
  and skips every one another machine owns, saying which machine holds it.
- Fixed by the operator: one launcher per hub plus one that starts every hub;
  beams are included; resuming interrupted runs is left to the orchs;
  nothing another machine owns is started; `.desktop` on Linux and `.cmd` on Windows, added and removed by lym.
