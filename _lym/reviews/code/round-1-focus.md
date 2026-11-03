---
round: 1
focus:
  - "internal/gateslot/config.go `ResolveCommandDir` against the gate's own run path (`WithDefaultDir`, `resolveRunDir` in internal/gatecli and every other reader of a gate `dir`): do they resolve an empty, absolute and relative dir identically, so the Goal's one base holds for tiers, build, capture and compare?"
  - "Every remaining consumer of a drift `CommandStep.Dir` or `AnchorDir` after `LoadConfig` stopped defaulting `dir` to `.` (internal/driftcli, internal/loomcli, battencli, side capture, `RunSideCapture`): does any still join onto the anchor, print, or compare `Dir` assuming it is non-empty?"
  - "The `WorktreeDir`/`AnchorDir` wiring in internal/driftcli/cli.go, internal/driftcli/wiring.go and internal/loomcli/driftwiring.go: is each request filled with the worktree root and the anchored directory, never swapped or left empty on a path that reaches `runStep`?"
  - "internal/configcli/hubwide.go `checkHubModule` and its callers: is `dirs.root` the prime's worktree root and `dirs.worktree` its anchor on every hub-wide `--set` and editor-save path, and does the before/after comparison keep a write allowed when the file already failed the same check?"
  - "Goal's Expected bullet 3 (`lym config <module> --set` and the config loader refuse a missing dir): does the diff's `CheckDirs`-on-write reading meet it for every module that declares a command dir, or is a module with a `dir` key left out?"
  - "Goal's Expected bullets for the Drift failure: does the `missingDirRefusal` text carry the resolved path, the `drift.yaml` key and the way forward `lym config drift --set <step>.dir=<dir>` from the prime then `lym drift capture` and `lym loom resume`, and does that text survive unchanged into the capture record's reason and the Drift-Await stop?"
  - "The notice change in internal/battencli/dobbynotice.go and wire.go: does shortening against the worktree root keep the full resolved path for a doubled dir, and does it change any other notice path the Goal did not license?"
  - "Template and doc text (internal/driftengine/template.yaml, internal/gateslot/template.yaml, doc.go files): does each template say, next to the `dir` key, which base it resolves against, and do the docs the repository's constraints document names for this kind of change land in the same diff?"
  - "Claims in _lym/landing/summary.md against the diff, notably the carry-over account for existing hub configs, the statement that the gate's run path is left alone, and the listed tests."
  - "Test economy in the new gateslot, driftengine and configcli tests: rows that repeat coverage another test already gives, per-helper tests the public `RunCapture`/`RunCompare`/dispatch tests already cover, and behavior added with no test."
---

Round 1 seed from the decision record's Goal and the range diff; the record's other sections are empty, so the Goal is the measuring stick.
