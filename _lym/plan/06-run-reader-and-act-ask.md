# Card 6 — run-reader-and-act-ask

**Create:**
- `plan:internal/pairteardown#RunReading` -> `type RunReading struct`
- `plan:internal/pairteardown#ReadRun` -> `func ReadRun(prime *lymcwd.Location, slug string) (RunReading, error)`
- `plan:internal/pairteardown#ActAnswer` -> `type ActAnswer struct`
- `plan:internal/pairteardown#AskAct` -> `func AskAct(prime *lymcwd.Location, slug string, aside runfsm.Asides) (ActAnswer, error)`
- `plan:internal/pairteardown#ActRefusal` -> `func ActRefusal(slug, rerun string, answer ActAnswer) error`
- `plan:internal/pairteardown#PublishBattenRecord` -> `func PublishBattenRecord(prime *lymcwd.Location, slug, msg string, push bool) (pushed bool, err error)`
- `plan:internal/pairteardown_test#TestReadRun` -> `func TestReadRun(t *testing.T)`
- `plan:internal/pairteardown_test#TestAskAct` -> `func TestAskAct(t *testing.T)`
- `plan:internal/pairteardown#TestActRefusal` -> `func TestActRefusal(t *testing.T)`
- `plan:internal/pairteardown_test#TestPublishBattenRecord` -> `func TestPublishBattenRecord(t *testing.T)`

**Uses:**
- `internal/pairteardown/doc.go`
- `internal/pairteardown#ReadBattenRecord`
- `plan:internal/runfsm#Records`
- `plan:internal/runfsm#Derive`
- `plan:internal/runfsm#Inputs`
- `plan:internal/runfsm#Act`
- `plan:internal/runfsm#Asides`
- `plan:internal/runfsm#Verdict`
- `plan:internal/shedrun#CancelLock`
- `plan:internal/fabricengine#Topology.TaskRecordsFile`
- `plan:internal/fabricengine#Topology.OriginParentHoldsTaskWork`
- `internal/fabricengine#Topology.TaskBranchOnOrigin`
- `internal/fabricengine#CommitAnchoredPaths`
- `internal/fabricengine#PushPairAnchored`
- `internal/battenrecipe#RowsBeforeRunShed`
- `internal/loomshed#NameFinalize`
- `internal/boardengine#IsEmptyStatus`
- `internal/boardengine#IsDoneStatus`
- `internal/boardengine#ReadOwners`
- `internal/pauserecord#ReadCoverage`

**Intent:**
Add the one reader that turns a run's tracked records into its D1 state, the one ask of the act rule every verb and loop makes, and the one publisher of the batten record.
They sit in `internal/pairteardown`, which already reads the batten record and the child's landing, and which every caller of the act rule imports:
`internal/battencli`, `internal/loomcli` and `internal/tendercli`, the last of which `battencli` imports, so the reader cannot live in `battencli`.

`ReadRun` reads tracked records alone, so a run whose `.lym` is removed reads the same state.
- The board entry, from the hub's local board: `Released` is a task entry with an empty status, `BoardDone` a done one.
- The batten record, from the prime's `_lym/shed/<slug>/status.json`, read tolerantly: its state, its row against `battenrecipe.RowsBeforeRunShed`, and its cancel and pause intents.
- For a finished cancel only, whether the task branch is still on origin.
- The child record, only for a batten record at Run-Shed or later: from the task worktree when the pair stands, else from origin's records branch through `Topology.TaskRecordsFile`.
  It is at Finalize or later when its row is `NameFinalize` or its history holds a done Finalize entry.
`RunReading` carries the derived `State`, the `runfsm.Records`, whether the pair stands, the batten record and whether one was found.
It also carries whether a live cancel holds `shedrun.CancelLock`, probed only for a cancelling run, and whether the run is past its parent-side merge.
The latter is read through `Topology.OriginParentHoldsTaskWork` for a landing run only.
This is the past-merge reading the carry-over finding readrun-pastparentmerge-no-pair asks for: it works with no pair here.

`AskAct` reads the run with `ReadRun` and fills `runfsm.Inputs`.
The owner is the effective owner of the local Bolt's owners through `hubgeom.ParentOf`, set only when it is another machine than `machinename.Name`, with the adopt subject beside it.
The pause coverage comes from this machine's pause record through `pauserecord.ReadCoverage`, and the facts come from the reading.
It reads no owner of a hand-over twice and reads no child once the owner hands the run over.
The hold, owed-push and code-tip inputs stay false; later parts fill them.
`ActAnswer` carries the `runfsm.Verdict`, the reading, the adopt subject and whether the covering pause is the hub's.

`ActRefusal` words the refusal of an answer that is not act, ending in a way forward and `rerun`:
- owned elsewhere: `lym adopt <subject>` here first, or the verb on the owner;
- a pause: `lym tender resume <slug>`, or `lym tender resume` for the hub's;
- paused by its intent: `lym tender resume <slug>`; paused by its shed state: `lym batten run <slug>` from the prime;
- cancelling with a live cancel: wait for it to end; cancelling with none: `lym batten cancel <slug>` from the prime to finish it;
- cancelled: `lym batten cancel <slug> --remove`, then release the task;
- landed: nothing is left to run;
- an owed push, a missing code tip and a hold: run again once origin answers, once the owner's push lands, or once the adopt's check is green.
No wording names `lym adopt --take-hold`.

`PublishBattenRecord` commits the prime's batten record path through `fabricengine.CommitAnchoredPaths`, with the run directory's path when the record is gone.
With push set it then pushes the prime's records side through `fabricengine.PushPairAnchored` and reports whether the push landed.
An unreachable origin is no error: it reports not pushed and leaves the commit for the next push.

`TestReadRun` is a tier 1 hubforge test that builds each state from tracked records alone, with every `.lym` cache absent.
It reads the same state again after removing `.lym`.
It covers a cancelled record whose task branch is gone from origin, which reads queued for a released task.
It covers a kept cancel read on a machine without its pair, which reads cancelled.
`TestAskAct`, tier 1, answers hand_over for a run another machine owns, including a landing past its merge.
It also covers a landing whose local parent branch holds the task's work while origin's fetched parent does not: that run reads as before its merge and answers wait under a pause.
`TestActRefusal` is a tier 0 table over every reason, asserting each way forward and that none names `--take-hold`.
`TestPublishBattenRecord`, tier 1, asserts the committed record reaches origin and that an unreachable origin reports not pushed.

**Commit:** 6: pairteardown reads a run's state from tracked records and asks the act rule
