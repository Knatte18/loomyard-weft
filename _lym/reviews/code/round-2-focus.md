---
round: 2
focus:
  - "internal/loomcli/wiring.go near the loomrecipe.HasEngine call for the DriftBaseline row: is the engine name passed as a named constant, consistent with how darnWriteEngine is declared in internal/loomcli/arm.go?"
  - "internal/driftengine/diffmode.go, the skip_file argument removal in the config migration: for a command such as sh -c 'docker run -e A={skip_file}' and for a backslash-continued multi-line command, is the rewritten command still balanced and free of stray escaped spaces?"
  - "internal/driftengine tests for openJudgeSession ordering, hub-only volatile_files and volatile_keys refusal, and the expect pass recording of volatile lists: does each behavior have a test that fails if the behavior is reverted?"
---
