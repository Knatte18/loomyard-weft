# Card 4 — reed opens an own-window strand in a detached window of its own

**Create:**
- `plan:internal/reedengine#unseenWindowOption` -> `const unseenWindowOption = "@lyx_unseen"`
- `plan:internal/reedengine#Engine.openOwnWindowLocked` -> `func (e *Engine) openOwnWindowLocked(s *Strand, name, group string) (string, error)`
- `plan:internal/reedengine#refuseWindowMove` -> `func refuseWindowMove(name string, current, next render.Display) error`
- `plan:internal/reedengine#TestOwnWindow_RealTmux` -> `func TestOwnWindow_RealTmux(t *testing.T)`

**Edit:**
- `internal/reedengine#validateDisplay`
- `internal/reedengine#Engine.launchStrandLocked`
- `internal/reedengine#Engine.updateStrandLocked`
- `internal/reedengine#Engine.killStrandPanes`
- `internal/reedcli#reedCLI.addCmd`
- `internal/reedengine/doc.go`
- `docs/overview.md`
- `internal/reedengine#TestAddStrandLocked`
- `internal/reedengine#TestUpdateStrandLocked`
- `internal/reedcli#TestStrandNamingAndList`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/reedengine#ownWindowOption`
- `plan:internal/reedengine#isOwnWindow`
- `plan:internal/reedengine#TmuxCmd.listWindowPanes`
- `plan:internal/reedengine#windowPane.livePane`
- `plan:internal/reedengine#Engine.listStrandAndWindowPanes`
- `internal/reedengine#strandLabel`
- `internal/reedengine#Engine.segmentColor`
- `internal/reedengine#Engine.setStrandPaneOptionsLocked`
- `internal/reedengine#paneIDsInSession`
- `internal/reedengine#alivePanePIDs`
- `internal/reedengine#newColdScratchEngine`
- `internal/reedengine#windowOfPane`

**Intent:**
D1, D4's no-focus rule, D7, D8 and D9: `own-window` becomes an accepted anchor, and its pane is created by `new-window -d` inside the one launch chokepoint every strand goes through.
Groups (D2) arrive in card 5, so in this card every own-window strand gets a window to itself, named by its role.

`validateDisplay` accepts `render.AnchorOwnWindow` beside the other three and hands it to `render.ValidateDisplay`, which refuses a negative `WindowSize` (card 1); the unknown-anchor message lists `below-parent|hidden|top|own-window`.

`launchStrandLocked`:

- The reconcile at its head is unchanged (card 3).
- For an own-window display it does not plan a split in window 0.
  It calls `openOwnWindowLocked(s, strandLabel(s.Name), "")`.
  A relaunch by `Resume` or `--if-absent` of a strand still bound to its dead own-window pane kills nothing, since D6 lets reed kill an own-window pane only on its strand's removal.
  The launch rebinds the strand to its new pane, so the old dead pane is left unbound, and card 3's reconcile ends its window, after D5's client select, once none of that window's panes is bound.
- Every other display splits in window 0 exactly as today.
- Both branches then share the existing tail: the title lock, `setStrandPaneOptionsLocked` (`@strand`, `@strand_color`), the pane-binary prelude with `LYX_STRAND_NAME` and `LYX_PARENT`, the launch script and the guarded send-keys, so `PATTERN-pane-binary-resolution` and `PATTERN-agent-name` hold for own windows by construction.
- Its doc comment says the launch either splits window 0 or opens an own window, and that both run the same tail.

`openOwnWindowLocked(s, name, group)`:

- Runs `new-window -d -P -F '#{pane_id} #{window_id}' -t =<session>: -n <name> -c <PaneCwd>` with no command argument, so the pane is the default login shell as for every strand, and `-d` keeps the client's current window and active pane where they are; nothing ever selects the window.
- An empty pane id or window id in the answer is an error, as `validateSplitCreatedNewPane` refuses a silent split.
- Then, by window id (never by name, since two windows may share one), it sets the window options `@lyx_own` to 1, `@strand_color` to the strand segment's tmux color (unset when the segment has none), and, everywhere but Windows, `@lyx_unseen` to 1, since psmux lacks `set-hook` and nothing would ever clear it.
  When `group` is non-empty it also sets `@lyx_group` to it (card 5 passes one).
  A failed option is logged at Warn and never fails the launch; a window that missed `@lyx_own` is merely never ended by reconcile, the safe side.
- `remain-on-exit` stays the global `on` pinned at boot, so an agent that dies leaves its window showing the dead pane (D5).
- It logs `reed: opened own window` at Info with the strand, window and pane, and returns the pane id.

`refuseWindowMove(name, current, next)` (D9), pure:

- When exactly one of `current` and `next` is `own-window` (a hidden strand surfacing into `own-window` included), it returns `strand %q cannot move between own-window and another anchor; way forward: lyx reed remove --name %s, then add it again with the new anchor`.
- When both are `own-window` and `Window` or `WindowSize` differs, it returns `strand %q cannot change its window group; way forward: lyx reed remove --name %s, then add it again with the new group`.
- Otherwise nil.

`updateStrandLocked` calls it after the unknown-guid check and before the hide check, so the strand's display is unchanged on a refusal.

`killStrandPanes` lists the session through `tmux.listWindowPanes` instead of `listStoredStrandPanes`, and maps the answer with `livePane()` before `paneIDsInSession` and `alivePanePIDs`.
Membership in this worktree's session stays the kill filter (R5-F4), now across every window, so removing an own-window strand kills its pane and, with the window's last pane, the window ends.
Card 6 adds the move of a viewer to window 0 and the group relayout.

`reedCLI.addCmd` accepts `--anchor own-window`: the switch admits it, the deferral message goes, the invalid-anchor message and the flag's help list `below-parent|hidden|own-window`, and the comment above the switch says the CLI vocabulary is below-parent, hidden and own-window.

`internal/reedengine/doc.go` gains an "Own windows" paragraph: an `own-window` strand gets a detached window named by its role, opened through the launch chokepoint, carrying `@lyx_own`, `@strand_color` and `@lyx_unseen`; it never takes focus; its dead pane stays until the strand is removed, or, once a relaunch leaves it unbound, until its window holds no bound pane; Resume reopens it detached; and `UpdateStrand` refuses a move into or out of it.
`docs/overview.md`'s reed bullet says a strand may also live in a detached window of its own (`--anchor own-window`).

Tests:

- `TestAddStrandLocked` drops the `OwnWindowAnchor` rejection row, adds `OwnWindowNegativeWindowSize` (`own-window`, `WindowSize: -1`) to the rejected rows, and its doc comment drops "the deferred own-window anchor".
  It gains a fake-tmux subtest, `OwnWindowEmptyNewWindowAnswerIsRefused`, on its own engine and `installFakeTmux`: `new-window` answers an empty line for an own-window add, the add returns the error, and no strand is recorded.
- `TestUpdateStrandLocked` replaces `OwnWindowAnchorRejected` with rows, each rejected with the display unchanged and the error containing `way forward: lyx reed remove`: `BelowParentToOwnWindowRejected`, `OwnWindowToBelowParentRejected`, `HiddenToOwnWindowRejected`, `OwnWindowGroupChangeRejected` and `OwnWindowSizeChangeRejected`; and `OwnWindowFocusOnlyChangeAccepted` (same group and size, `Focus` toggled) is accepted.
  Its doc comment names the window-move refusal.
- New `TestOwnWindow_RealTmux` in a new file `internal/reedengine/ownwindow_integration_test.go` (`//go:build tmux`, package `reedengine`, with a file comment), one cold session from `newColdScratchEngine` serving ordered subtests:
  - `OpensDetachedNamedByRoleWithoutMovingTheViewer`: after `Up` and a window 0 strand, the session's current window and active pane are read; an own-window `AddStrand{Role: "explore", Segment: "review", Cmd: "sleep 300"}` leaves both unchanged; the strand's pane is in a window other than the strands' window, named `explore`, carrying `@lyx_own` 1, `@strand_color` orange and `@lyx_unseen` 1; `Status` reports it live, present and own-window; and window 0's pane list is unchanged.
  - `DeadPaneStaysBoundAcrossLaterOps`: an own-window strand with `Cmd: "exit"` goes dead; across a later window 0 `AddStrand` and a `Status`, it stays bound and reported not live and present, while a window 0 strand with `Cmd: "exit"` added beside it has its dead pane killed and binding cleared by the next op.
  - `OrphanOwnWindowIsEndedAndAPlainWindowIsKept`: a window opened by hand with `@lyx_own` 1 and a pane no strand holds, and a second window opened by hand without it; the next `AddStrand` ends the first and keeps the second.
  - `OpenWindowOfTheSameNameLeavesTheOwnWindowAlone`: `OpenWindow("explore", ...)` returns `Existing` false with a window id other than the own window's, and the own window and its pane are still there.
  - `UpdateStrandRefusesAMoveOutOfOwnWindow`: `UpdateStrand` to `below-parent` fails naming the way forward, and the pane stays in its own window.
  - `ResumeAfterAServerRestartReopensDetached`: `kill-server`, then `Resume`; the live own-window strand from the first subtest is relaunched into a fresh window other than the strands' window, the session's current window is still not that own window afterwards, and an own-window strand whose own `Cmd` touched its done-when marker and then ran `exit` (standing in for D12's chain whose self-removal failed) stays bound to its dead pane until the restart and is then dropped instead of relaunched.
  - `RemoveEndsTheOwnWindow`: `RemoveStrand` of the own-window strand kills its pane and its window is gone from the session listing.
- `internal/reedcli#TestStrandNamingAndList`'s `TaskWorktreeRolesParentResumeAndRemove` subtest also runs `add --role explorer --anchor own-window --cmd "sleep 300"` and checks `status` names `tst:<slug>:explorer`, and runs `add --anchor sideways` and checks the error lists `own-window`.
- `cmd/lyx/testdata/test-budget.yaml`: raise `internal/reedengine` by one for `TestOwnWindow_RealTmux`.

**ImpactSummary:** Opens the own-window anchor end to end in reed and `lyx reed add`; window 0 strands keep the same split, tail and kill path, and every edited member keeps its signature, so no caller changes.

**Commit:** 4: reed opens own-window strands in detached windows of their own
