# Lens review

Every lens fork returned a report; each is saved beside this file as `lens-review.lens-<lens>.md`.
Duplicates across lenses are merged onto one line naming every lens that raised them.

- row-validity 1 + guard-reach 1, BLOCKING, fixed as proposed: card 2's desktop row with a non-empty mapping group now expects `ErrInvalid` naming `buttons.task.label` and "not in the desktop template", and only the empty-mapping row expects "different shape". Card 2's Intent states which `ValidateMachine` branch each shape reaches.
- coverage 1, BLOCKING, fixed as proposed: card 8 adds `button` to `internal/reedcli#Command`'s `PersistentPreRunE` skip list, and `TestRunCLI_Switch_RefusesBadFlagsAndNeedsNoRepository` gains a row that runs `button` with every flag, no repository and a missing socket, expecting exit 0 and no output.
- row-validity 2 + guard-reach 2, MEDIUM, fixed as proposed: card 2's reed local-override row asserts `"buttons.task"`, the unit `ValidateLocal` refuses.
- row-validity 3, MEDIUM, fixed as proposed: the Warn-asserting tests and subtests in cards 3, 4, 5, 6 and 8 run without `t.Parallel`, each with a comment naming the logger's process-global output.
- row-validity 4 + forced-edits 1, MEDIUM, fixed as proposed: card 9 states that `ShellPinsFollowTheOptionPins`'s `wantLast` gains the button-command unset between the follow-command and build-stamp unsets.
- row-validity 5, MEDIUM, fixed as proposed: card 5 names `assertSessionButtonsOnce`'s no-comma check and anchors it on `nolist align=right`.
- row-validity 6 + coverage 3 + coverage 8, MEDIUM, fixed as proposed: `TestRunButtonVia` gains rows for an unresolvable executable on a confirm entry, a spawn error from `run`, a spec value that is not JSON, a range suffix that is not a number, and a failed `@lym_strand_name` read.
- coverage 2, MEDIUM, fixed as proposed: card 9's tier2 recorder also records whether `LYM_STRAND_NAME` and `LYM_PARENT` are set, and the test asserts neither is.
- coverage 4, MEDIUM, fixed as proposed: card 2 asserts the whole D8 set (both groups, every label, run and confirm), and a subtest reads both template YAML files from the package directory and checks that each `buttons` node equals that set.
- coverage 5, MEDIUM, fixed as proposed: every `TestReedGeometry` row asserts `Worktree` is the anchor path, including the subpath-anchored task row, where it differs from the worktree root.
- row-validity 8 + coverage 6, MEDIUM, fixed as proposed: card 7 adds a launch row for an empty name expecting the unset, and a row whose `@lym_strand_name` write fails while the launch still succeeds.
- coverage 7, MEDIUM, fixed as proposed: the subpath-anchored row of `TestReedGeometry`, which has no origin record, reads `ButtonPlaceTask`, and `TestGeometryOverRealHub` gains a step where `Beam` returns the wrapped parent-read error over an unparseable origin record.
- guard-reach 4, MEDIUM, fixed as proposed: `TestRunButton_RealTmux` gains a `{strand}` entry run once with the client on a pane carrying `@lym_strand_name`, which fills the name, and once on a pane without it, which runs nothing and shows the agent-pane message.
- way-forward 1, MEDIUM, fixed as proposed: card 8's agent-pane message names reattaching with `lym reed attach`, the step whose pre-flight sets the name; the row asserts that wording.
- way-forward 2, LOW, fixed as proposed (the first of its two options): reed's Warn source label names both files that can hold the entry, `reed.yaml (the hub base or the worktree file)`, since the node is read after the layers are stacked.
- way-forward 3, LOW, fixed as proposed: card 6's missing-beam Warn names `lym adopt <beam>` and the re-render at the next attach, and its row asserts `lym adopt`.
- way-forward 4, LOW, fixed as proposed: an unresolvable executable on the confirm path shows a message naming the label and the lym log in the run directory.
- way-forward 5, LOW, fixed as proposed: the bad-strand-name message tells the operator to click the button again from the agent pane.
- way-forward 6, LOW, fixed as proposed: an unreadable, non-JSON or empty spec shows a message naming `lym reed attach`, which renders the spec.
- guard-reach 3, LOW, fixed as proposed: every `display-message` text doubles `%` as well as `#`, and a row pins `100% done` under the label `a#b`.
- coverage 9, LOW, fixed as proposed: the re-pin row asserts every argv of `statusFormatPinArgvs()` before the first `bind-key`, and a new non-parallel subtest pins that a failed format re-pin is logged and the bindings are still re-pinned.
- coverage 10, LOW, fixed as proposed: `TestPlaceButtons` gains `#`, `%` and a control character in both fields, `{strand}` in `confirm`, a mapping `prime` group falling back to `beam` with a Warn (now stated in card 3's Intent), and an absent group giving no Warn.
- coverage 11, LOW, fixed as proposed: card 6 adds the row `ButtonGroupMappingPasses` to `TestUp_BootValidation`, pinning that Up's boot validation never refuses on a button.
- row-validity 7, LOW, fixed as proposed: card 8's Uses names `internal/reedengine#installFakeTmuxOn` instead of `installFakeTmux`.
- row-validity 9, LOW, fixed as proposed: card 9's stub is a task whose `Beam` returns the temp directory, so the recorded `pwd` pins the run-directory rule.
- row-validity 10, LOW, fixed as proposed: the vague `#` row is replaced by the label `a#b` in the failed-run message.
- forced-edits 2, LOW, fixed as proposed: card 5 rewrites `TestPinGeometryOptionsLocked`'s doc comment and `//testtiming:keep` reason to three formats and status 3.
- forced-edits 3, LOW, fixed as proposed: card 5 lists `TestBar_LineTwoShowsTheCheckMarkBeforeTheClock` and rewords its stale error text.
- forced-edits 4, LOW, fixed differently: the proposal was to list `internal/shell/shell.go`'s header, but listing that file beside `internal/shell#Shell` on card 1 is refused by `redundant-file-target`. Card 1's Intent now names the header change in the file it already edits.
- forced-edits 5, LOW, fixed as proposed: card 2 lists `internal/reedengine#ConfigTemplate` and names `buttons` among the plain-literal keys in its doc comment.
- forced-edits 6, LOW, fixed differently: the proposal was to list `geometry.go`'s header, but listing that file beside `internal/reedengine#Geometry` on card 4 is refused by `redundant-file-target`. Card 4's Intent now names the header change.
- forced-edits 7, LOW, fixed differently: the proposal was to list the comments of `cmd/lym/stencilseedgate_test.go`, but a `_test.go` path under Edit is refused by `edit-test-file`. Card 8, which already lists `cmd/lym#TestSkipStencilSeed_HonoursTheAnnotation`, now names the file header and doc comment change in its Intent.
