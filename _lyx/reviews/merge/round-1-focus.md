---
round: 1
focus:
  - "internal/yamlengine/set.go and layers.go: the parent added list-valued set handling (a per-round model list written as a list) and the task added scalarSetTag for null and int leaves. Settle whether a list key, a null-template leaf and a spelled null each reach the branch both sides assume, in SetValues and in SetSparse, and whether the merged doc comments still describe the code."
  - "internal/battencli/wire.go and internal/battenshed/innerrun.go: the parent moved the terminal open into OpenTerminalOnce, called from the create row, and made PushSeed a detached spawnPairPush. Settle whether the task's arbiter-eligible stop hold, ArbiterStop wiring and nil-default in NewInnerRun still sit consistently with the parent's changed wire and deps, and whether the PushSeed contract in deps.go holds for the task's seed path."
  - "pattern/PATTERN-fabric-git.md (Add pushes nothing, callers use the detached SpawnPairPush): settle whether any call site the task added or changed pushes a new pair synchronously or assumes an origin branch after Add."
  - "pattern/PATTERN-no-denied-recovery.md (the systemctl sleep verbs joined the closed deny list) and pattern/PATTERN-daemon-wakeups.md (the hibernate watch joined the unbounded-wait list): settle whether any stencil, spec or Go string literal the task added names a denied form, and whether any wait loop the task added belongs on the wakeups list."
  - "contracts/stencils/claims_test.go, contracts/stencils/orch/orch-template-role.md and contracts/specs/refusal-spec.md: both sides edited the orch stencil claims and the refusal spec. Settle whether the merged claims, stencil text and spec rows agree, including the driver stencil's arbiter wording against the orch stencil's new hibernate wording."
  - "cmd/lyx/testdata/test-budget.yaml and docs/overview.md: settle whether the merged budget rows match the packages' top-level test counts after both sides added tests, and whether the module table in docs/overview.md carries the new modules from both sides."
  - "Test economy: settle whether the tests the task added to internal/battencli/lifecycle_integration_test.go or internal/yamlengine overlap the parent's new coverage there, and whether the interaction of the hold with the create-row terminal open and pair push has any coverage."
---

Seeded from the merge's overlapping files and the three changed pattern files.
Each entry names where to look and the question to settle, with no verdict pre-stated.
