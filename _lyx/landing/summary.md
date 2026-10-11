# Pane injection never merges with an operator draft; batten survives one

Text lyx types into a Claude pane no longer merges with, submits or wipes what the operator is typing, and no longer closes a menu the operator has open.
A draft in a parked driver's pane now delays batten's resume instead of failing batten, and a halted batten whose child loom run is done is flagged and closed by the orch's after-landing routine.

## What changes

Before, a verified send spent about a second typing into the pane (Escape, 300 ms, text, settle reads, Enter) after a single pre-write check.
The first Enter was never checked against the box, `boxHoldsSentText` matched only the leading 48 characters, and `ClearSession`, `CompactSession` and `ReloadPlugins` typed on the caller's earlier probe.
`judgeReadiness` read a screen with no input box as ready, so the leading Escape could close a picker.

Now every text write into an agent pane (start prompt, skill load, `Runner.Send`, gate sends, `/clear`, `/compact`, `/reload-plugins`, `/color`, and reed's `/rename` repair) goes through one path:

- **Pre-write guard.** One fresh reading must show the session ready, no attached client on the target pane active within `operator_quiet_s` (new `reed.yaml` key, default 5 s), and an empty input box. A failed reading holds the write; the idle wait keeps polling, and on timeout the send fails with `ErrSessionBusy` naming the reason, with nothing typed.
- **Atomic paste-submit.** A new `reedengine` op runs one tmux invocation: fill a uniquely named buffer, `paste-buffer -p -d`, `send-keys Enter`. The leading Escape and the 300 ms settle are gone from `claudeengine`, which declares that it takes the paste path.
- **Enter only on an exact match.** After the paste, an Enter that lyx sends itself goes out only when the box holds exactly the sent text or Claude Code's collapsed-paste placeholder. Any other box content gets no Enter, no clear and no edit; the send fails wrapping `ErrSessionBusy`. The new `ErrSentTextInDraft` marks the case where our text sits inside the operator's draft.
- **Pending-send record.** A send that may have left text in the box (`ErrSentTextInDraft`, `ErrSubmissionNotLanded`) records its text and events offset in `pending-send.json` in the run directory. The next send of the same text, once the box reads empty, returns landed without typing when a turn start past the offset exists, and types afresh otherwise.
- **Unreadable box is held.** `judgeReadiness` holds a screen with no input box for an engine that implements `InputBoxReader`. An engine without it is unchanged.

Callers treat a held write as "later":

- `orchengine`'s watcher treats `ErrSessionBusy` from any typed step as nothing typed, keeps the phase and notice, retries next tick, and does not count it toward `maxConsecutiveTickErrors`. A held idle-phase notice records its reason in `State.Stuck`, visible in `lyx orch status`.
- `lyx loom start` and `lyx loom resume` refuse a busy or drafted driver pane with a new kind, `driver_busy` (row added to `contracts/specs/refusal-spec.md`), naming the draft and the way forward. The park marker stays, and the resume report path is now fixed once per park (in `internal/shedrun`) so a rerun types the same line.
- `battenshed` maps `driver_busy` to a sentinel and retries it like `ErrChildNotParked`, at `noticeProbe` cadence, queuing one orch notice per stall. The notice never quotes the draft.
- `lyx batten status` adds a flag when the batten run is failed, blocked or paused and its child loom run is done, naming `lyx batten run <slug>`.
- The orch role stencil's `## After landing` section gains a first step: check `lyx batten status <slug>` and run `lyx batten run <slug>` when it is not done.

`PATTERN-pane-text-injection` records the invariant: text goes into an agent pane only through reed's atomic paste-submit behind shuttle's pre-write guard, and no Enter goes into a box that does not hold exactly the sent text.

Closes the board entry `pane-inject-safety`; it has no GitHub issue.

## Design decisions

- reed writes, shuttle decides. reed gains the paste-submit and an activity query and stays provider-blind; shuttle owns readiness, activity and box state. This keeps the `creel → shuttle → reed → tmux` layering that a later queue module will wrap.
- An operator-activity query that errors is logged at Warn and does not hold the write, so a tmux that cannot answer `list-clients` cannot stall every run. The empty-box check and the Enter guard still stand.
- `Start` keeps a held or foreign-box failure of its skill load or prompt delivery as a died start. A foreign-box failure kills the pane at once, with any draft in it.
- No new queue module: every other caller already has a retry loop. `Runner.Interrupt`, `Runner.Inject` and the engines' interrupt and model-switch sequences stay as they are, since they deliberately bypass idleness.
- Known residual risks, accepted: a key pressed in the milliseconds between the pre-write capture and the tmux invocation can still merge; an operator paste older than `operator_quiet_s` but inside the confirm window can pass the placeholder check; `client_activity` also moves on mouse and resize events; the pending-send record drops one retry when the operator deletes our text and sends the rest; a slash command left pending and then submitted by the operator is typed twice on retry.
- batten's retry of a busy driver has no deadline, as the not-parked retry has none.

## To check by hand

Not run in this session:

- The `tmux`-tier tests (`internal/reedengine/paste_integration_test.go`, `activity_integration_test.go`) were vetted with `-tags tmux` and not run.
- The `llm`-tier test `internal/shuttlecli/smoke_paste_test.go` pins whether Claude Code submits a plain text and `/color` on an Enter right after a bracketed paste end, and that a long paste collapses into the placeholder. It was compiled with `-tags llm` and not run. Run it; if it fails, flip `pasteSubmitsOnEnter` in `claudeengine` to switch to the no-Enter variant.
- The assumption that `#{pane_id}` in a `list-clients -t <session>` format resolves to each client's active pane is verified only by the tmux-tier test above.

Open findings to review:

- Resume line stability (`internal/loomcli/driverresume.go`): the report path is stored once per park beside the marker so reruns type the same line and the pending-send record matches; check that no path leaves a rerun with a different line.
- A not-landed send that may have landed has no record when it fails before the paste-submit; check that `ErrSubmissionNotLanded` records only after a paste.
- `internal/shuttlecli` reaching an unexported constant: check that the `llm`-tier smoke test depends on no unexported `shuttleengine` or `claudeengine` identifier.
- The foreign-box error should name the draft: check that the `driver_busy` message and the `ErrSessionBusy` reasons quote the box text, as `ErrSentTextInDraft` does in `internal/shuttleengine/send.go`, while the batten notice stays free of it.
