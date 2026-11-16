# A transition pushes its records before the run acts, and a takeover entry in the run's records hands the run over

`transition-push-takeover` · Task · feature, batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut · running · Plan-Write

Decision record: [Sketch](decision-transition-push-takeover.md)

Filed from the Sketch `adopt-build-resplit`.

Takes: runfsm's Judge, Aside and Inputs, whose PushOwedOffline and CodeTipMissing nothing fills; pairteardown.ReadRun, the one reader of a run's inputs, and EndSubjectProcesses; the detached commit-and-push of lym tender pause, lym tender resume, lym batten cancel, the pull-request record and the fixer attempt counts; fabricengine's PushPairAnchored (code side first today), coalescePushSides, coalesceFlagged and respawnForeignFlags, the teardown-lock respawn, pushclassified's rejection path, anchorchange's push and ArchiveOwnerTips; statuscommit's per-transition spawn; boardengine's Sync, SpawnSync and owner commit; landingshed's Finalize; machinename.Names; PATTERN-test-only-code holds, so every new function has a production caller in this part
Delivers: a shed transition that takes a pauserecord.Admit admission, commits its records naming its code tip and pushes records side first and code side after before the next producer starts; PushPairAnchored and coalescePushSides sending records first and no code side after a failed or rejected records side; the run intents, the PR record and the fixer counts on the same synchronous push, the offline verbs acting here at once and owing their push; the per-transition detached push, the teardown-lock respawn and the pending-flag respawn retired; an owed push retried on a backoff behind the gate and read by ReadRun into runfsm's PushOwed (renamed from PushOwedOffline) with its last error carried in the inputs, the verdict, the refusal and lym beam status; synchronous Bolt writes for a run claim, a landing's done and an owner; Finalize pushing the parent branch before its transition and its board done, and reading the owner before its merge and each later push; a rejected records push that reads origin's record against the name set its caller hands in and hands over on another machine's takeover entry, archiving local tips and the local parent merge, removing the local pair through pairteardown's local mode when the caller runs outside the pair, writing the hand-over mark under the hub's .lym and skipping a takeover a mark names; ReadRun setting CodeTipMissing, the code-tip wait, its 30-minute grace and the leased push and reset of an owed tip
Exports: the takeover entry type and its reader, called in the rejected-push path, and its test writer in internal/testkit; a pair-push entry that takes the set of names reading as this machine, so the registry part passes its names in later without changing fabricengine's signature again; the hand-over mark's path in internal/lymdirs and its reader; runfsm's PushOwed input with its last error; the code-tip grace constant

The part changes these packages:

- internal/fabricengine (about 1340 changed lines)
- internal/battencli (about 300 changed lines)
- internal/landingshed (about 250 changed lines)
- internal/sheddriver (about 200 changed lines)
- internal/loomcli (about 200 changed lines)
- internal/boardengine (about 150 changed lines)
- internal/statuscommit (about 100 changed lines)
- internal/tendercli (about 100 changed lines)
- internal/pairteardown (about 100 changed lines)
- internal/runfsm (about 60 changed lines)
- internal/testkit (about 60 changed lines)
- internal/shedengine (about 60 changed lines)
- internal/beamcli (about 60 changed lines)
- internal/fabriccli (about 40 changed lines)
- internal/tenderengine (about 40 changed lines)
- internal/pauserecord (about 40 changed lines)
- internal/pullrequest (about 30 changed lines)
- internal/lymdirs (about 10 changed lines)
- pattern (about 30 changed lines)
