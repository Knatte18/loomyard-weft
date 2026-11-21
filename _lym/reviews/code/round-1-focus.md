---
round: 1
focus:
  - "internal/runplan/postbatch.go and internal/discussionparser/append.go: does a dropped Create skip only the target the decision names, and do record-batch and recover-batch run the same checks and give the same way forward?"
  - "internal/loomcli/decision.go `--drops-create`: is every refusal path (filed run, unknown target, empty value) handled, and does the written `Drops Create:` line round-trip through `DroppedCreates`?"
  - "internal/websterengine/verifygate.go: does the stuck reason name the full verify-gate report path in every branch, and does the drift way forward name `lym drift amend` correctly?"
  - "internal/burlerengine/roundcheck.go and internal/shedadapters/roundcontext.go: does an `--amends-plan` decision exempt only the cards it names from the changes-file hunk rule, and can `origin: webster` admit a finding it should not?"
  - "internal/runplan/conflict.go and internal/shedadapters/webster.go: is the halt reason written when the run goes stuck, cleared when it is done, and is a read or write failure handled and not silently dropped (survivors at webster.go:64 and :71)?"
  - "internal/shedadapters/burlerreview.go lines 360-383 and internal/shedrecipe/entries_burler.go: what do the surviving mutants change, and which behavior has no covering check?"
  - "internal/websterengine/render.go and the Webster stencils: does a later parent decision supersede an earlier one in the master prompt and the implementer job text, with record order preserved?"
  - "Scope: the diff touches weaverengine, testkit/envkit and burlercli/validate.go; check each change against the amended board entry, and check that the auto rebaseline after a fix (#170 bullet 2) is not implemented."
---

The answer key is the decision record's `## Goal` plus its `Added after Discussion` entry, which wins where they differ.
The goal body is corrupted, so the entry's three-part scope and the #178 part 1 inclusion are the measuring stick.
