---
round: 2
focus:
  - "internal/gatecli/cardfixer.go newStrandRig: does any run path reach the notifier set by SetNotifier, given the card fixer is started by gatefixer.Spawn and polled by gatefixer.Await rather than Run.Wait, and do the gatecli doc.go and _lym/landing/summary.md claims about held-turn mail match that answer?"
  - "internal/loomcli/arm.go loomCLI.newShed: does a test pin the notify_parent_on activation, the empty-list short cut, the record and status paths passed to the row-notice wrapper, and the refusal of an unknown row name?"
  - "internal/loomcli/rownotice.go and its docs (contracts/orchrole/orch-role.md, internal/loomcli/doc.go, shedrun.RowNotice, internal/loomengine/template.yaml): does the once-per-visit wording match verdictCount's behavior when a row is re-called after an Awaiting escalation or a resumed Stuck halt?"
  - "internal/loomcli/rownotice_test.go: do the tests pin the wrapper's stated contracts, namely no retry after a failed post, notice before the row's call, unchanged pass-through of the inner pointer and error, and warn-only on a missing status file?"
  - "internal/shedrun/paths_test.go TestPathConstructors and internal/loomengine/config_test.go TestLoadConfig_Loads: do they carry rows for RowNoticeRecord and for the notify_parent_on decoding, as their sibling constructors and parent keys do?"
---

The board entry asks to mail the parent when the run reaches a named producer or state, while the diff builds recipe rows only.
That narrowing is the parent's call and no focus entry asks the reviewer to settle it.
