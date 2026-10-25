---
round: 3
focus:
  - "Full pass refused as owned elsewhere: in decision-record.md D3, D4 and D7, check what dobby does when a full pass (which runs the resume's origin reconcile, where resumeRefusal returns the owned-elsewhere refusal before the driver steps) exits non-zero because another machine took the run over. Settle whether the run is retried forever as a failing pass, or whether the owner pause, parent-lock release and owner notice naming lyx adopt happen exactly once."
  - "Spawn hold with a persisted owner: in D4's spawn-hold paragraph, check whether a run held on a standing ownedBy can still reach the once-per-probe owner re-check that clears it (holdSpawnWhileCovered's in-loop ownerStep), and settle whether a run adopted back or whose owner pause lifted is respawned by a later pass."
  - "Continue pass and PATTERN-owner-machine: the declared change says a continue pass dobby schedules into a run it already drives needs no origin verdict, while the first pass after a dobby start or a machine wake, every pass while a full pass is owed, and every hand pass is a resume and gets one. Check that D4's continue-pass paragraph, its flag bounds and the PATTERN changes section agree with that text, and settle whether any path lets a resume skip the verdict."
  - "Owed full pass: in D4, check what clears the owed full pass, in particular a full pass that resumed on the local Bolt because origin was unreachable. Settle whether a takeover that happened while the network was down is still judged against origin afterwards."
  - "One check per pass: in D4, check which round of the child-wait loop a pass's one check is, and that the owner trigger fold, owner step, watched-set refresh and idle lock release all run in it; check that the owner trigger's process-local sleep watch is accounted for in the memory-record inventory, together with the cross-pass respawn count and the api-error revive marks and counts from D4 and D9."
  - "Ownership of adopted runs: in D2 and D3, check who drives an adopted member task whose parent beam another machine owns, and that the matching acceptance criterion names the case."
  - "Release of draft tasks: in D5, check every board write that can leave a task with an empty status, including an upsert that turns a note into a task and an explicit empty status in an upsert or merge, against D5's claim that only an explicit release does so."
---

Targets the open round-2 keys without prescribing fixes.
