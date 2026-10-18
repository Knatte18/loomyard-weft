---
round: 2
focus:
  - "In internal/creelengine/deliver.go, check how the scheduler snapshot and markNoticeRead treat a program or EXTERNAL notice that a concurrent Store.Read holds reserved: is it typed or journaled read at most once, and does a row in TestScheduler_Reminders in deliver_test.go fail if that breaks?"
  - "In internal/creelengine/deliver_test.go, check whether a notice from a sender other than the hold owner stays untyped under a hold, and whether a due inject is typed before a notice, and whether a row would fail if either broke."
  - "Check that the post command's help in internal/creelcli/cli.go and the creel entry in docs/overview.md describe both delivery forms: agent-strand mail announced by a reminder and read with lyx creel read, and EXTERNAL or program mail carried in full in the typed line."
---

The focus restates the places where the round-1 fixes land, so a fresh reviewer settles whether they hold.
