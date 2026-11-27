---
round: 1
focus:
  - "internal/shedadapters/multillm.go, mapOutcome's chair-timeout branch with internal/seatengine/engine.go join and resume.go: on every path that reaches this Stuck reason (fresh Run, Resume of a live table, a step a writer or required advisor ended, a failed chair stop), has the chair been stopped, so that 'was stopped' is true?"
  - "The same reason's way forward, traced through MultiLLMProducer.Call, the seat probe and shuttle's handling of a stopped run: does lym loom resume after this halt always start the row in a fresh session, with no re-attach or provider resume of the stopped one?"
  - "Comments and docs that describe a timed-out chair or this reason (internal/shedadapters doc.go and multillm.go, internal/seatengine, repository docs): is each accurate after the change, including the new package-doc sentences and the line about every such Stuck reason beside them?"
  - "Scope against the board entry in the decision record: compare the diff with each item under 'What to do', including the alternative and the 'Consider' item, and check the change description's claims about them and its Merge danger door and blast radius against the diff."
  - "internal/shedadapters/multillm_test.go and the commit trailers: do the changed expectations still check what their row names say, does any other test or caller depend on the old wording, and does the Root-Cause trailer name the defect the diff removes?"
---

The diff is one commit that rewords the chair-timeout reason, its tests and the package doc.
The wording's value rests on it matching runtime behaviour, so the focus puts the code paths behind the two claims first.
