```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked every bound, guard, guarantee and claim about existing code in the decision record.

**Claims attacked**
- D1, the goal claim "a gap is a bug, and a test fails on it", with a parity test that covers only seats in `seatengine`'s tables: **broken** (F1).
- D1, a manifest read back from the launch catches a dropped delivery: held.
- D2, the allowlist bound "admits a Go literal that never reaches a Codex session": **broken** (F7).
- D2, the scan skips the orch places: held.
- D3, the edit-rule guard exempts the engine copies: held. The test exempts any stencil whose `path.Base` is `edit-directive`, and `EngineBlockName` already ends in `edit-directive`, so the guard is not weakened.
- D3, "an assist seat gets the full directive too": **broken** on a hub that edited its seeded engine copy (F6).
- D4, only `lym shuttle run` with a prompt file marks a spec hand-written: **broken** twice (F2, F3).
- D4, the orch pinned to Claude under a Codex default engine: held as stated.
- D5, "settled the launch-length risk" from the launch code: **broken** (F5).
- D5, an empty set omits the key, and the value is one TOML string: held (`tomlString` escapes newlines).
- D7, an older lym refuses an `agents-prompt` store: held as the stated bound.
- D7, the hub reconcile renders into every live worktree: held. The walk visits every code worktree pair (`internal/hubreconcile/doc.go:35-37`).
- D8, plugin hooks run "the way Claude Code runs it", each with a timeout: **partly broken** (F8).
- D9, the cap is computed above the total size of the files read: held.
- D10, the verb starts no session and writes nothing: held as a bound; whether shared Prepare code writes files is plan-level.
- D12, the nudge goes to the model through the hook's `reason`, "as creel's Codex hook does": **broken** (F4).
- D12, `PreToolUse` denies `request_user_input`: **unproven** (F9).

**Findings**

- **F1 — MEDIUM — design — D1 and the Goal.** The parity test covers only `seatengine` seats, so a Codex gap in any other producer fails no test.
  Counterexample: webster's render paths call `editdirective.Directive(stencilsDir)` and build their own prompts (`internal/websterengine/render.go:236,382,456`). So do treadle (`internal/treadleengine/judge.go:72,103`, `targeting.go:44`), sampler (`internal/samplerengine/spec.go:78`), the tackler (`internal/sheddriver/tacklerprompt.go:60`) and PR-Review (`internal/landingshed/prsession.go:124`). None of these is a `seatengine` seat. A dropped Codex delivery in any of them passes D1's test, which contradicts the Goal's "a test fails on it". D1 needs to cover every producer that builds a launch spec, or state the bound.

- **F2 — MEDIUM — design — D4's bound.** `lym shuttle run --prompt "&lt;text&gt;"` would be refused on every engine, Claude included.
  The verb takes exactly one of `--prompt` or `--prompt-file` (`internal/shuttlecli/run.go:112-118`), and its help shows the inline form (`run.go:74`). An inline prompt is just as hand-written as a prompt file. D4 marks only the prompt-file spec, and refuses an unset prompt engine.

- **F3 — MEDIUM — design — D4's bound.** Nothing holds "only `lym shuttle run` marks a spec hand-written".
  The scan fails only on a spec literal that sets neither the prompt engine nor the mark. A producer that sets the hand-written mark passes the scan and skips the refusal; its manifest then calls a Claude-worded prompt "hand-written". The bound needs a holder, for example the scan also failing on a hand-written mark outside `internal/shuttlecli`.

- **F4 — MEDIUM — design — D12.** Delivering the nudge through a `PostToolUse` block `reason` makes Codex treat the nudged call as failed.
  `internal/shuttleengine/codexengine/doc.go` says: "A block after a tool call makes Codex read that call as failed, so the session may run it once more". It also says Codex reads a block reason as a failed tool call unless its instructions say `[creel]` lines are mail, and the nudge is not a `[creel]` line. So a nudge after a successful call can cause a rerun, which is more calls, the opposite of its purpose.
  Claude's nudge uses non-blocking `additionalContext` (`internal/shuttleengine/claudeengine/batchingnudge.go:297` → `settings.go:89`). Codex 0.164.0 takes `additionalContext` with `systemMessage` on `post_tool_use` without a block (`docs/research/creel/creel-midturn-trust-2026-10-28/report.md:198-206`, `creel-hook-channels-2026-10-28/report.md:26`). D12 picks the channel with a documented side effect and does not address it.

- **F5 — LOW — design — D5, and support-log Turn 3.** The launch-length risk is called settled on evidence that does not settle it.
  The launch line is written to a script that the pane sources (`internal/reedengine/doc.go:386-388`), so tmux's command limit does not apply. But `developer_instructions` reaches `codex` as one argv element, and Linux limits one argument to 128 KiB. That value carries the user's global file, every `CLAUDE.md` above the root and the skill listing (D8), plugin outputs, and `@` imports expanded up to five hops. A global file that imports large documents exceeds the limit and the launch fails with E2BIG. No bound or fallback is stated.

- **F6 — LOW — design — D3.** "An assist seat gets the full directive too" fails on a hub whose seeded engine copy was edited.
  `stencilstore` never overwrites a seeded file whose hash no longer matches (`pattern/PATTERN-stencil-ownership.md:18`). A hub that edited `engine/claude/edit-directive.md` or `engine/codex/edit-directive.md` keeps its thin copy, while the shared stencil it fell back to is gone. D3's bound names only the loss of the shared stencil's edits.

- **F7 — LOW — design — D2's bound.** The allowlist entry's claim that a literal "never reaches a Codex session" is free text that nothing checks.
  Counterexample: an allowlisted literal later reaches a Codex prompt through a new caller, and the scan still passes. The bound holds only by review, and the record does not say so.

- **F8 — LOW — design — D8's bound.** "Run at `Prepare` the way Claude Code runs it, each with a timeout" leaves the timeout and the hook's input unstated.
  Claude Code passes a SessionStart hook a stdin payload with `session_id` and `source`; no Codex session id exists at `Prepare`. A hook that keys state on `session_id` behaves differently. With no timeout value, a hanging hook stalls every Codex launch for an unstated time.

- **F9 — MEDIUM — design — D12, deferred fact.** The `PreToolUse` deny of `request_user_input` rests on a matcher name nobody verified.
  The twill spike found Codex's `PreToolUse` payload names `spawn_agent` as `collaborationspawn_agent` (`docs/research/codex/codex-twill-spike-2026-10-24/report.md:127`). D12's evidence is only that the binary holds the string `request_user_input` and that `PreToolUse` fires for non-shell tools. If the payload names the question tool under a prefix, the deny matches nothing, and a failed hook fails open (`codexengine/doc.go`). So whether the deny works is left for the implementer to find out.

**Tool calls**
1. `internal/seatengine/prompt.go` and `table.go` `stencilEngine`: to confirm which seats get `EngineDirective` (D3, D1); the manifest lacked the seat branch.
2. A grep of `hubreconcile` callers and heddle render functions: to test D7's reconcile claim; not in context.
3. `internal/hubreconcile/doc.go:25-50` and its heddle grep: to see what the reconcile renders today (D7).
4. A grep for `PromptFile` and `PromptPath` across production: to find other entry points to D4's hand-written mark.
5. `internal/shuttlecli/run.go:105-145`, and callers of `lym shuttle run`: to check D4's prompt-file bound against the verb's flags.
