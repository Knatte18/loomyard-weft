---
round: 2
focus:
  - "internal/shuttleengine/send.go `clearBoxBeforeCommand`: does the C-u clear run only after an operator-activity check made immediately before the key and a box read showing blank or unreadable, so no operator text typed after the idle read is deleted?"
  - "internal/shuttleengine/send.go `confirmPasted` for a send that starts no turn: is `ErrCommandTakenAsPrompt` returned only for a turn start whose instruction matches the pasted text, so an unrelated turn start past the paste cannot make creel requeue and retype a command that ran?"
  - "internal/shuttleengine/session_test.go `checkPastedSessionCommand` and internal/creelcli/delivery_test.go `mapDeliveryError` table: does each behavior added to the clear and the confirmation, and the pass-through of `ErrCommandTakenAsPrompt`, have a row without duplicating an existing test?"
---

Focus covers the command-paste clear and confirmation paths in `internal/shuttleengine/send.go` and the tests that pin them.
