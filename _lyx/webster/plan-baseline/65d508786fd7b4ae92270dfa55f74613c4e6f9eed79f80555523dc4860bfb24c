# Card 12 — orch-resume-context

**Edit:**
- `internal/orchcli#orchCLI.resumeContextCmd`
- `internal/orchcli#newTestCLI`
- `internal/orchcli#TestResumeContext_PrintsHookJSONPerPhaseAndFailsWithoutMark`

**Intent:** Build the orch's `resume-context` hook JSON through the `ResumeContextRenderer` capability instead of `claudeengine.SessionStartContext` (D9).

`resumeContextCmd` takes the set's default engine from `c.engines`, the set card 7 fills, since the orch spec names no engine, and asserts `shuttleengine.ResumeContextRenderer` on it.
It renders the resume text first, as today, then:
a default that does not resolve, or an engine without the capability, fails the verb with the JSON error envelope and writes no delivery mark, the verb's existing failure path, so the watcher types the pointer instead;
the capability error reads `orch: the engine "<name>" cannot render session-start hook context`.
The unresolved default is a fail-loud wiring guard with no per-row test: `shuttleengines.New` refuses a `cfg.Engine` naming no registered engine (card 5's `build`), so no production input reaches it, and card 1's `TestEngineSet_Lookup` already pins the lookup's error.
Otherwise it writes `ResumeContext(text)`'s bytes as today.
`internal/orchcli/resumecontext.go` no longer imports `claudeengine`, and with it no production file outside `internal/shuttleengines` and the engine's own directory imports a concrete engine.
The orch stays Claude-only (`engine-model-choice` scope), so this changes nothing for the shipped engine.

Tests: `newTestCLI` sets `engines` to `shuttleengine.NewEngineSet("claude", ...)` over `claudeengine.New()`, so the existing rows render through the Claude capability unchanged.
`TestResumeContext_PrintsHookJSONPerPhaseAndFailsWithoutMark` gains a row whose receiver's `engines` is `shuttlefake.EngineSet(&shuttlefake.Engine{})`, an engine without the capability: exit 1, an error envelope naming `fake`, and no resume mark.

**ImpactSummary:** One orchcli verb body and its test fixture; the hook output for the Claude engine is byte-identical.
