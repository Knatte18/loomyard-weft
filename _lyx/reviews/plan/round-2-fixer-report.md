# Plan round 2 — fixer report

Every finding's premise holds against the code: `attach_test.go:549` scripts `relevantWindowFixture` and a failed `list-clients`, and `newAttachTestEngine` answers an unscripted `display-message` with `$0|4321|1700000000`;
`attach.go:278` carries the stale sentence;
`windowsize.go:162` pins `status 2` on the reed session only;
the `click` helper (`bindings_integration_test.go:157`) clicks at `y = bindingsClientRows - 1 + row`;
`bindings.go:74` builds the one `MouseDown1Status` click argv, so the `bind-key` count cannot tell `followJump` apart.

## Changes

- F1 (`02-attach-follow.md`): added `internal/reedengine#TestAttachArgv_MultiClientWarning` to **Edit:**;
  the Intent has `ListClientsError_WarnsAndAppendsNoSelection` script `fake.answerFormat("#{@lyx_follow}", "off", nil)` and keep its no-selection expectation as the failed-listing-with-follow-off row, beside the follow-on row `FailedClientListingWithFollowOnSelectsTheRelevantWindow`;
  the test's doc comment sentence about a byte-identical argv is rewritten to hold only while follow reads off or no window is relevant.
  The ImpactSummary names the one changed subtest.
- F2 (`02-attach-follow.md`): added `internal/reedengine#Engine.warnMismatchedClientsLocked` to **Edit:** with no re-sign arrow;
  the Intent rewrites its doc comment's "which is what drops the attach window selection from the argv" to "which AttachArgv's selection gate reads".
- F3 (`06-follow-bindings-button.md`): the weird-named session row runs `set-option -t =<name>: status 2` after `new-session -d -s <name>` and before `switch-client`, with the reason.
- F4 (`06-follow-bindings-button.md`): `BindingsAreIssuedAfterTheOptionPins` also asserts the issued `MouseDown1Status` `bind-key` argv equals the one from `bindingArgvs("", "", false)`;
  D7's fallback paragraph now names that assertion as the row reaching `pinBindingsLocked`'s choice, and says why the count alone cannot.
- F5 (`03-follow-run-op.md`): took the review's first option.
  The test attaches four clients and detaches none; the waits are `waitForClientAttached` for the first and `waitUntil` counts 2, 3 and 4 for the rest.
  `RunStartSelectsTheOverviewThenEachNewAgentForEveryClient` attaches the first and second;
  `ASecondAttachMovesTheFirstClientOnlyWhileFollowIsOn` attaches the third (count 3) and fourth (count 4), asserting on every attached client.
  Every attached client is put on the shell by one `select-window -t <shell window>`, since a session's clients share its current window.

`lyx loom validate-plan` reports ok.
The review's own `--overlay` runs covered F1 and F2; F3 to F5 change no target.

## Disputed

None.

## Deferred

None.
