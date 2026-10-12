# Card 7 — dialog-notice

**Create:**
- `plan:internal/shuttleengine#permissionNoticeBound` -> `const permissionNoticeBound = 60 * time.Second`
- `plan:internal/shuttleengine#dialogLatch` -> `type dialogLatch struct`
- `plan:internal/shuttleengine#dialogNotice` -> `func dialogNotice(strandName string) string`
- `plan:internal/shuttleengine#Run.watchDialog` -> `func (run *Run) watchDialog()`
- `plan:internal/shuttleengine#Run.foldDialogAsks` -> `func (run *Run) foldDialogAsks(parser SessionSignalParser)`
- `plan:internal/shuttleengine#Run.notifyDialog` -> `func (run *Run) notifyDialog() bool`
- `internal/shuttleengine/dialog_test.go`

**Edit:**
- `internal/shuttleengine#RunState`
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine#TestParseSessionSignals`
- `internal/shuttleengine/claudeengine/doc.go`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine#readBox`
- `internal/shuttleengine#Run.notifyHeld`
- `internal/shuttleengine#holdNotice`

**Intent:**
D6: an autonomous run whose pane shows a permission or input dialog lyx cannot answer notifies its parent once the dialog has stood for a bound, so a fork's dialog no longer stalls a run unseen until its deadline.
Every new member lives in a new file, `internal/shuttleengine/dialog.go`, whose header says it holds the pending-dialog latch and its notice.

`permissionNoticeBound` is 60 seconds: long enough that an operator answering a dialog at once sees no notice, short enough to surface a stall well inside any run's timeout.

`dialogLatch` is the per-Wait, in-memory latch state, kept on a new `Run` field: the events-file cursor its signal reads resume from, whether the latch is set, its stamp time, the cursor after the batch holding its latest folded ask, and whether its notice was sent.
`RunState` gains `DialogNotifiedOffset int64` with JSON key `dialogNotifiedOffset,omitempty`, beside `NotifiedOffset` and documented like it, so a held-turn-end notice and a dialog notice never suppress each other.

`Run.Wait` starts the latch's cursor at the larger of `RunState.PromptOffset` and `RunState.DialogNotifiedOffset` on every path, an attached run included, and calls `watchDialog` on each tick right after `settleDeferredHold`.
The loop's classification, outcomes, deadline and liveness check are unchanged, and the latch adds no wait of its own (`PATTERN-daemon-wakeups`) and finalizes nothing (`PATTERN-completion-signal`).
The file header of `wait.go` gains one provider-neutral sentence on the latch beside the hold notice.

`watchDialog` does nothing for an interactive run, a runner without a notifier, or an engine that lacks `SessionSignalParser` or `InputBoxReader`.
`Spec.QuietHold` and `Spec.HoldDeferral` neither suppress nor defer it, since no peer can answer a dialog.
Otherwise it calls `foldDialogAsks`, then, while the latch is set, captures the pane through reed and reads it with `readBox`:
- a capture error or a pane too short to draw the box is a failed read, which changes nothing and is retried on a later tick;
- a read showing an input box clears the latch at any age, so a later ask opens a new episode;
- a read showing no input box, with the latch at least `permissionNoticeBound` old and its notice not yet sent, calls `notifyDialog`, which marks the notice sent when it returns true.
After the notice, the reads go on each tick until one shows an input box.

`foldDialogAsks` reads the events file from the latch's cursor through `readEventsFrom` and the parser, advancing the cursor by the consumed count.
An ask is a `SessionSignalAsk` that is not also an event (`SessionSignal.Event` false): the engine's permission and input-dialog signals, never the question tool's ask, which `ParseEvents` already yields.
The first ask while the latch is clear sets it, stamped with the signal's `At`, or the run clock's time when `At` is zero; a further ask while it is set opens no new episode and keeps the stamp.
The latch is independent of the session-state fold: a later turn end or turn start, the main session's or a fork's, never clears it.
When an ask folds into a latch whose notice was sent, the cursor after that batch is persisted as `DialogNotifiedOffset` (under `recordMu`, as `notifyHeld` persists), so one dialog's later ask events, such as a `permission_prompt` Notification after a `PermissionRequest`, or a second fork's dialog folded into the episode, are never rebuilt into a latch on reattach; a failed write is logged at Warn.

`notifyDialog` persists the latch's latest folded offset, the cursor after the batch holding the latest ask folded into the episode, as `DialogNotifiedOffset` before calling the notifier, restoring the old value and returning false when the write fails, as `notifyHeld` does.
It then sends `dialogNotice` with the strand name, or the strand guid when `RunState.StrandName` is empty; a notifier error is logged at Warn and still counts as sent, so the notice is never repeated.
`dialogNotice` is one line starting "Shuttle notice:", naming the strand, saying its pane shows a permission or input dialog lyx cannot answer, and giving the way forward: the operator answers the dialog in the pane, reached with `lyx reed attach` in the run's worktree.
It carries no agent-written text; any added later goes through `delimitedAgentText`, as `holdNotice`'s does.
It names no provider and no receiver, since board notes plan to change who receives notices.

The bound of this edge: at most one line per latch episode to the parent that already receives hold notices; it never types into the asking pane (`PATTERN-orch-pane-single-writer`, `PATTERN-pane-text-injection`) and never changes an outcome.

`dialog_test.go` holds `TestWait_DialogNotice`, one table over `Wait` under an injected clock in `hold_test.go`'s shape, with `readinessEngine` as the engine and a fake reed of its own whose pane shows a caret input box or no box as each row's per-tick script sets it, and can fail a capture.
"ASK" lines are the asks, as `sessionFakeEngine` reads them.
Its rows pin:
- an ask with no input box sends exactly one notice once the bound has passed and none before it, with `DialogNotifiedOffset` persisted before the notifier call;
- an ask followed by a turn end with outstanding tasks, with no input box, still sends the notice;
- an input box shown before the bound sends nothing;
- an interactive run sends nothing, a runner without a notifier sends nothing, and a `QuietHold` run still sends the notice;
- two asks within the bound send one notice;
- an ask, an input box at 10 s and a second ask at 55 s send no notice before 60 s after the second ask;
- a notified dialog answered (an input box) and then a second ask sends a second notice once the bound passes on the second ask;
- a `HoldDeferral` run's pending held-turn-end notice is still sent when its deferral ends after a dialog notice was sent;
- a failed capture sends nothing on that tick and the notice follows on a later tick that reads no box;
- a further ask after the notice advances the persisted `DialogNotifiedOffset` past it;
- a run whose record's `DialogNotifiedOffset` is before a pending ask, as a reattach finds it, sends the notice once, and one whose `DialogNotifiedOffset` is past every ask, two asks for one dialog included, sends none.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/shuttleengine` by one for it.

`TestParseSessionSignals` gains a row: a `PermissionRequest` payload carrying a top-level `agent_id`, a fork's, reads as an ask that is not an event, like the main session's.
`claudeengine`'s `doc.go` says which of its hook events are the non-question asks the dialog latch reads: `PermissionRequest` and the `permission_prompt`, `elicitation_dialog` and `elicitation_url_dialog` Notifications, a fork's included.
`internal/shuttleengine/doc.go` gains a provider-neutral paragraph on the dialog notice beside the hold notice: when the latch sets and clears, the bound, the notice's content, its persisted offset and its reattach rule.
`docs/overview.md`'s shuttle row says an autonomous run notifies its parent once per dialog its pane shows past the bound.

**ImpactSummary:** Adds one notice path to every autonomous run's wait loop and one optional run.json field; classification, outcomes and the hold notice are untouched, and the shuttleengine budget row and one claudeengine signal row follow.
