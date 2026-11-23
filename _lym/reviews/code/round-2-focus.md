---
round: 2
focus:
  - "Check whether any package other than fabricengine spells the records namespace root as a literal in code, help text or package docs, in internal/fabriccli/fabric.go and internal/fabriccli/doc.go."
  - "Check what `lym batten run` prints when FetchPairTips fails on a records store that lost its fetch refspecs, in internal/battencli/catchup.go (fetchRunForCatchUp and catchUpFailure): does the operator see the refspec way forward or an unreachable-origin message?"
  - "Check that each records-layout read in internal/fabricengine/codetipsettle.go (fastForwardWeft, FetchPairTips, ReturnToPreviousTransition) has a test that fails when the read is deleted."
  - "Check RefuseHubWithoutBolt in internal/fabricengine/clone.go for every path that names a clone command: one-repository join, one-repository seed killed before the board was pushed, a lost-refspec layout error, and ForceBootstrap."
  - "Check whether a failure of PlainArchiveTagsOnOrigin can turn `lym fabric cleanup --remote --apply` into an error after deletions have run, in internal/fabriccli/fabric.go."
---

Focus entries restate what round 1 left open; they are places to look, not verdicts.
