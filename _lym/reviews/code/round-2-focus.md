---
round: 2
focus:
  - "The amended board entry expects a beam member's parent-review asks, notices and escalations to go to the beam's orch, and that orch's answer to be accepted. Check both halves in internal/orchroute (walk.go#FromPair, Asker, live.go#Live), orchcli's NotifyParent, NotifyParentHalt, ParentStrand and ParentOrchRecorded, and the dobby's hubgeom.NearestLiveOrch call in internal/beamcli/dobbydeps.go: where does a member's message go when the beam orch's liveness probe answers false, and does a test hold that case?"
  - "Which verbs admit a caller as a run's parent, and does each admit every orch the answer key allows? Cover lym drift amend (internal/driftcli/amend.go#checkParentCaller) beside decisionCallerRefusal and decisionStoreVerb in internal/loomcli/decision.go and parentAnswerCallerRefusal in internal/loomcli/parentreview.go. For each, what happens to a caller the nearest-live parent check would admit when reading the orch chain fails?"
  - "contracts/stencils/parent/parent-directive-handback.md tells the tackler and arbiter that a handed-back stop reaches the named parent. Does the name those prompts carry (hubgeom.ParentOrchName, the loom ParentName) match where the halt notice goes, both when the beam orch reads as down and when the recorded parent pair is gone?"
  - "What happens in internal/battencli/drive.go (loadedTacklerDeps, loadedArbiterDeps) when hubgeom.ParentOrchName returns an error, and does any test fail if the parent name stops reaching the tackler or arbiter prompt in batten or loom wiring?"
  - "Is the real decisionOrchChain in internal/loomcli/decision.go exercised for both a beam's project record and a task record, or only replaced by fakes?"
  - "Do docs/overview.md, internal/hubgeom/doc.go, the verbs' help texts and _lym/landing/summary.md describe the caller rules and routing as the code now has them, and does each added doc.go sentence document only its own package?"
---
