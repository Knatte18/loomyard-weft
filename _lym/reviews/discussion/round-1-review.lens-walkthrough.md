```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case and checked the end state against the decision's purpose.

**Decisions walked**
- D1, case: the darn writer's seat. One spec per engine goes through both engines' `Prepare` with a faked probe and hook runner, and the two manifests are compared. Purpose met.
- D2, case: the discussion stencil's Step 2 passage (Agent tool, `subagent_type`, "subagent passage of your system prompt"). It moves into an engine block, and the Codex copy uses `spawn_agent` without a model. Purpose met.
- D3, case: a webster producer under a Codex model. It passes the engine to `editdirective.Directive` and renders the Codex copy. The edit-rule test's `path.Base == "edit-directive"` check already matches both engine copies. Purpose met.
- D4, case: a producer that forgets to set the prompt engine. `Prepare` refuses it. A bare `lym shuttle run` is marked hand-written. Purpose met.
- D5, case: a Codex run with guards, creel and standing passages. These items go into `developer_instructions`, which is the same channel the creel research's Codex case used for the trust sentence. The prompt file holds only the prompt. Purpose met.
- D6, case: a six-lens fork review on Codex. The cap is 16 and the twill spike showed the key raising the slot count. Purpose met.
- D7, case: `lym-batching` moves to `agents-file`, and the reconcile re-renders the worktree. Both agent files carry it. Purpose met.
- D8, case: the scribe plugin's SessionStart `echo`. The session preloads `scribe:prose`, so the scribe plugin is on and lists every skill it has (`scope.go:64-67`). The listing therefore carries `scribe:conversation`'s SKILL.md, and the hook output asking to load it can be acted on. Purpose met.
- D9, case: a session at the worktree root, which has no tracked CLAUDE.md. The override file and the cap hold. The repo's only tracked nested instruction file is a research fixture's `AGENTS.md`. Purpose met.
- D10, case: `lym shuttle manifest` for the darn writer seat. The manifest comes from `Prepare`'s output, but `Prepare` writes files and, under D8, runs hooks. **Purpose not met as specified** (finding 1).
- D11, case: an edit to the `codex-engine-trial` entry. Purpose met.
- D12, case: a Codex session that makes a sliced read through shell. **Purpose not met** (findings 2 and 3).

**Findings**

1. **MEDIUM**, `design`, D10 (decision-record.md:144-148, with D1 at :27, :34 and D8 at :124).
   - Summary: the verb "writes nothing and runs no plugin hook" yet "shares the test's code". That code builds the manifest by reading back what `Prepare` wrote and ran.
   - Mechanism: Codex `Prepare` writes the prompt file into the run directory (`internal/shuttleengine/codexengine/codexengine.go:130`). Under D1 it also writes the manifest record, and under D8 it runs every plugin SessionStart hook.
   - Gap: the record never says how the verb gets a read-back manifest without those writes and runs. It names no scratch run directory and doesn't say whether the faked hook runner is used outside tests.
   - Result: the decision's bound and its mechanism contradict each other, and the plan writer gets no way through.
2. **MEDIUM**, `design`, D12 (decision-record.md:162).
   - Summary: D12 sends the nudge line through a PostToolUse block `reason`. Codex reads a block after a tool call as a failed call, so the session may run that call again.
   - Evidence that this happens: `internal/shuttleengine/codexengine/doc.go` ("A block after a tool call makes Codex read that call as failed, so the session may run it once more") and `capabilities.go:51-56`.
   - Result: a nudge meant to cut calls can add a repeat call.
   - Claude's nudge does not block: it uses `additionalContext` (`claudeengine/batchingnudge.go:297`, `settings.go:89`).
   - Codex already takes a non-blocking channel: the creel research saw Codex `post_tool_use` with `additionalContext` reach the model (`docs/research/creel/creel-hook-channels-2026-10-28/report.md:26`, `creel-midturn-trust-2026-10-28/report.md` case 5). D12 cites creel's block path without weighing that side effect.
3. **MEDIUM**, `design`, D12 and its criterion (decision-record.md:162, :209).
   - Summary: "a Codex call chain that breaks the batching rules" has no defined rule set on Codex.
   - Claude's judgment is built on Claude's own tools: the matcher is the tools a breach is made of (`claudeengine/batchingnudge.go:87`), and a sliced `Read` with `file_path` is the core breach (`batchingnudge.go:133`, `:511`).
   - Codex has no Read tool. Its reads and edits are shell commands and `apply_patch`, so the hook's matcher and breach rules have no stated mapping.
   - Result: neither the walkthrough nor the acceptance test can decide what Codex chain earns a nudge.

**Tool calls**
- `scope.go` outline: to check whether D8's skill listing covers a plugin skill the session never preloaded. My context had no session-scope semantics.
- `git ls-files` for CLAUDE.md and AGENTS.md: to give D9 a concrete nested-file case. My context had no list of tracked instruction files.
- Grep of codexengine `Prepare` and the heddle override render: to confirm what Codex `Prepare` writes, for the D10 walkthrough. My context lacked `Prepare`'s body.
