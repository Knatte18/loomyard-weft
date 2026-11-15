# Every transition pushes its records before the run acts, and a takeover entry in the run's records hands it over without replay

`push-before-act-takeover` · Task · feature, batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle

Decision record: [Sketch](decision-push-before-act-takeover.md)

Depends on: [`run-fsm-tracked-records`](design-run-fsm-tracked-records.md)

Filed from the Sketch `adopt-redesign`.

Takes: runfsm's Act inputs and the batten record from run-fsm-tracked-records; fabricengine's PushPairAnchored, SpawnStatusPush, teardown-lock respawn, pushclassified's rejection path and ArchiveOwnerTips; boardengine's Sync, SpawnSync and pushOwnerCommit; statuscommit and sheddriver's records commit; landingshed's Finalize; runproc's End for a slug
Delivers: a shed transition that takes a gate admission, commits its records with the code tip it was made with and pushes records side first, code side after, before the next producer starts; an owed push retried on a backoff and shown in lym beam status, which tender's offline pause and resume intents also ride; the per-transition detached push and its teardown-lock respawn retired; synchronous Bolt writes for a run claim, a landing's done and an owner; Finalize pushing the parent branch before its transition and board done; a rejected records push that reads origin's record and hands over on another machine's takeover entry, archiving local tips and the local parent merge; the code-tip wait and its grace
Exports: the takeover entry type and its reader, tested with kit-written entries; the owed-push and code-tip facts runfsm's Act reads; fabricengine's records-first PushPairAnchored and its hand-over result

The part changes these packages:

- internal/fabricengine (about 1000 changed lines)
- internal/battencli (about 250 changed lines)
- internal/sheddriver (about 200 changed lines)
- internal/loomcli (about 200 changed lines)
- internal/landingshed (about 200 changed lines)
- internal/boardengine (about 150 changed lines)
- internal/statuscommit (about 100 changed lines)
- internal/shedengine (about 60 changed lines)
- internal/beamcli (about 60 changed lines)
- internal/fabriccli (about 40 changed lines)
- internal/tendercli (about 40 changed lines)
- internal/tenderengine (about 40 changed lines)
- internal/pauserecord (about 40 changed lines)
- pattern (about 30 changed lines)
