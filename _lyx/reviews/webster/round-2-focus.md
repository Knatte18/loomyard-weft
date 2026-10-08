---
round: 2
focus:
  - "Refusals reachable through a webster verb that this branch adds or changes: in internal/webstercli/reset.go (the uncommitted-path read after the run record is archived) and internal/websterengine/runlevel.go (the overviewFrameHash error returned from Run, and through restampBaseline). Settle whether each message names a way forward that matches the state the operation left, whether contracts/specs/refusal-spec.md has a row for it in the same commit, and whether a test reaches it."
  - "Transient refusals in internal/websterengine/merriambase.go (merriamBaseError), internal/websterengine/recoverbatch.go (uncommitted-path listing) and internal/websterengine/rebaseline.go (overview index-only read). Settle whether each has its own refusal-spec row or is covered by a row whose wording actually includes that kind of read."
  - "internal/batcher/config.go MigrateConfig: settle where the orientation key is inserted when both retired keys master_base and startup_context are present, in either order and with either last, and whether TestMigrateConfig covers that combination."
  - "Long help text in internal/webstercli/run.go: settle whether any dash or separator in the reworded sentence can be read as a flag."
---

The focus points at sites the branch's diff touches; it caps no severity and states no verdict.
