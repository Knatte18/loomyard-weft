---
round: 1
focus:
  - "Publish failure `TestLoomResumeReconcilesThenRefusesWithoutLiveDriver` (`internal/loomcli`): the log shows `TempDir RemoveAll cleanup ... bares/weft-bare/objects/tmp_objdir-incoming-*: directory not empty`, plus a tmuxkit leak notice for key `lyx-p61b6f42`. Settle whether the cause is a detached push (or other background git process) still writing into the bare weft repo when TempDir cleanup runs, and whether the branch's detached-push skips (commits 4d34dffe, 94bc0800, fbca8658) cover this test's resume verb."
  - "Read the merge-in diff `860b0cd9^1..860b0cd9` for changes to `internal/reedengine/tmuxfloor.go`, `internal/reedcli/watchdog.go`, `internal/toolfloor` and `internal/testkit/tmuxkit` that interact with `loomcli` resume tests: does the merged-in code spawn or leave a tmux server or push process this test does not register or wait for?"
  - "The tmuxkit leak notice is a separate symptom in the same run: check whether the branch's 76abcca8 change (reed capability-probe socket exemption) matches the leaked key's origin, or whether this test leaks a server through an unregistered key."
  - "Test economy: the branch's new `loomcli` smoke and resume tests (`smoke_resumearming_test.go` and neighbours) against existing coverage of the same verbs; name the covering test before flagging."
---

Seed focus for round 1. The Publish failure is the one BLOCKING item the directive requires; the remaining entries point at likely causes and at the rubric's test-economy dimension.
