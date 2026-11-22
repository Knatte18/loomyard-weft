---
round: 1
focus:
  - "internal/hubgeom/tasksetting.go, internal/dobbyengine/tasksetting.go: does the lookup read only each worktree's own file, never the merged value, and walk worktree, parent beams, prime, hub default layer, template default in that order, with a lower value overriding in both directions?"
  - "internal/dobbyengine/tasksetting.go:48 and landnotice.go:170,173,191 (surviving mutants): is the changed constant or dropped statement behavior the diff adds or a contract its doc comment states, and can a test assert it?"
  - "internal/landingshed/pullrequestmode.go, finalize_pr.go, prreview.go, internal/loomcli/landingdeps.go: is pull_request fully moved onto the lookup, with the board field and the required and field-clause modes gone, and on, off and merge kept as values?"
  - "internal/dobbyengine/landnotice.go, view.go, loop.go, starts.go: is land_notice moved onto the lookup with the per-task mode gone and the values on and off, and does an unreadable or invalid value fail the way the old path did?"
  - "internal/boardengine/store.go (foldLegacyBeamFields, setBeamSettingIfAbsent) and internal/dobbyengine/reinterpret.go: does the one-time carry of notify_on_land and pull_request into run_settings keep every old meaning, never overwrite a value already set, and run once?"
  - "internal/boardengine/board.go, task.go, runlock.go, raddlefile.go, replay.go: does removing the board fields leave any caller, rendered output, payload key or run lock path that still reads or writes them?"
  - "pattern/PATTERN-config-layers.md, pattern/PATTERN-landing.md, internal/configreg, internal/dobbyengine/template.yaml: does each meaning change carry the reinterpretation the pattern asks for, and do the docs and template match the new values?"
  - "Tests: the TestEntryDates and TestLookupTaskSettings_RefusesAValueTheKeyDoesNotTake contained-candidates, plus any test the diff loosened or deleted in boardengine, landingshed and dobbyengine."
---

Focus covers the answer key's four asks: the one lookup, the two moves onto it, the one-time carry with reinterpretation, and removal of the old board fields.
