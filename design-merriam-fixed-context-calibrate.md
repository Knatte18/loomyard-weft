# Re-measure the Merriam fixed context with tools/tokencount

`merriam-fixed-context-calibrate` · Note · enhancement, webster

From webster-dead-ends PR-Gate (2026-10-08), moved out of `binary-change-and-rules` at its Discussion.

`merriamFixedContext` (`internal/websterengine/merriambase.go`) is a constant from one measurement of the system prompt and tools, so the computed Merriam base drifts silently as Claude Code changes.
A calibrate step in `tools/tokencount` re-measures it.
