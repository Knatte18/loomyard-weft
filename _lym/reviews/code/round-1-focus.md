---
round: 1
focus:
  - "D5 in `internal/shuttleengine/codexengine/codexengine.go` and `developerinstructions.go`: does the `developer_instructions` order match the record, is an empty set omitted, and does the 128 KiB refusal measure the quoted argument and name its size, largest item and way forward?"
  - "D7 in `internal/heddleengine` and `internal/hubreconcile`: the diff shows no change to the shipped `lym-batching` and `lym-plan-edits-before-gate` entries; check whether they already sit on `agents-file` and name no Claude tool, and whether `agents-prompt` renders into both prompt targets, is refused by `lym heddle show` with the way forward, and is re-rendered by the hub reconcile for every live worktree."
  - "D8 in `internal/shuttleengine/claudeengine/usersetup.go` and `pluginhook.go`: does each hook get the stdin input, env (`CLAUDE_PLUGIN_ROOT`, `CLAUDE_PROJECT_DIR` as `req.WorkDir`) and timeout the record sets; is a failing or timed-out hook left out with a warning; is the child waited on and killed (PATTERN-child-processes); does an unresolvable config dir drop only the parts that need it, with one warning and no error?"
  - "D8/D9 import expansion in `internal/claudeimports`: relative to the importing file, five hops at most, none inside a code span or block, cycles and unreadable paths handled; does `internal/heddleengine/importrecord.go` record only in-worktree files, and does the event matcher re-render on them?"
  - "D9 in `internal/shuttleengine/codexengine/instructionfiles.go`: does `project_doc_max_bytes` exceed the total size of the files on the root-to-workdir path, does `AGENTS.override.md` carry both files with imports expanded, and are the other CLAUDE.md files named correctly (below the workdir, or beside an `AGENTS.md`)?"
  - "D6 in `subagents.go`, `config.go` and `template.yaml`: effort from agents.yaml, `max_subagent_threads` default 16, `plugin_hook_timeout` default 10, and Codex subagent text that uses Codex terms and none of D2's closed terms."
  - "Seam layering: `internal/shuttleengine/usersetup.go` and `internal/shuttleengines/shuttleengines.go` against PATTERN-shuttle-provider-seam; does `codexengine` import `claudeengine` anywhere, and are the hook runner and sandbox probe injectable seams of Prepare?"
  - "Docs and PATTERN: do `HEDDLE.md`, `PATTERN-heddle-instructions.md`, `PATTERN-leaf-packages.md` and `PATTERN-shuttle-provider-seam.md` carry what the plan's declared changes list, and do the changed `doc.go` files match the code?"
  - "Test economy: `TestBuildLaunchCmd`, `TestLoadConfig_Template` and `TestPrepare_ExpandsHomeInWritableRoots` are reported contained; compare their assertions with the covering tests before filing; also check the new tests for stencil-prose assertions and for added behavior left uncovered."
---

Seeded from the decision record's decisions D5–D9 and the diff stat of the review range.
The D7 entry is a question because the stat lists no shipped-entry file, so the entries may have moved before the range.
