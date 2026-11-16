```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I tried to break each claim and bound in the decision record and the support log.

**Findings**

1. **MEDIUM — D2: the scan misses Claude wording that Go code writes into prompts.**
   - The scan only covers `contracts/stencils/` and `contracts/specs/`.
   - Go code outside the engines also writes Claude-only text into prompts, through the markers it fills:
     - `internal/loomengine/prompt.go:23,30` fills the discussion `mode_rules` value with "Never call the `AskUserQuestion` tool".
     - `internal/driftengine/judge.go:31` fills the judge's `mode_rules` with "never call the `AskUserQuestion` tool".
     - `internal/loomcli/parentreview.go:391` builds a prompt that names "a Bash tool timeout".
     - The messages in `internal/planparser/verifymodule.go` say "no fork may run raw" and point at "the gate rule in the agent's system prompt".
   - Counterexample: a Codex Discussion seat in autonomous mode gets the AskUserQuestion wording, and the scan stays green.
   - Fix: D2 also covers text that Go writes into markers. Either that text gets engine copies, or the scan reaches these Go literals.

2. **MEDIUM — D1: "removing any one Codex delivery fails the test" can be broken.**
   - D1 says the manifest is built by the same function `Prepare` builds the launch from, and the test compares manifests.
   - Counterexample: Codex `Prepare` stops writing `AGENTS.md` into the home, or drops `developer_instructions` from argv. The manifest function still lists those items, so the parity test passes.
   - The claim holds only if the test reads what was actually delivered: the launch argv, the files written in the home and the run directory.
   - Fix: say that D1's test reads the launch's actual artifacts. Otherwise drop the "removing any delivery fails" clause.

3. **MEDIUM — D4: a forgotten producer gets through as a "hand-written" prompt.**
   - The bound lets through any spec whose prompt engine is not set. So a production producer that never sets the field is not refused. D4's whole purpose was to turn a missed producer into a refusal.
   - Counterexample one hop past what the record cites: `orchSpec` builds a model choice with no engine (`internal/orchcli/start.go`). Any producer that leaves the engine empty falls back to shuttle.yaml's default engine. Under `engine: codex` it launches a prompt rendered for Claude, unrefused, and its manifest says only "hand-written".
   - Fix: only the `lym shuttle run` prompt-file path may leave the field unset. That entry point marks the spec as hand-written, and `Prepare` refuses an unset field from anywhere else.

4. **MEDIUM — D8: one shared Codex home serves a global file that differs per session.**
   - Claude's global file sits under `CLAUDE_CONFIG_DIR` or else `~/.claude` (`internal/shuttleengine/claudeengine/scope.go`, `doc.go:650`).
   - lym's Codex home is shared by every hub on the machine, so there is a single `AGENTS.md` there.
   - Counterexample: board entry `claude-account-pick-simple` (T1, draft) picks a Claude account per worktree, and so a config dir per worktree. Two `Prepare` calls in parallel with different sources race on one file. A Codex session started between them reads the other account's file. A `Prepare` whose source is absent deletes the file another session's launch has just written.
   - The race goes beyond "under the home's existing file lock": the lock orders the writes, but a session reads whichever write came last.
   - Fix: send the text per session through `developer_instructions`, or state the bound and name T1.

5. **MEDIUM — D7: a store with an `agents-prompt` entry breaks an older lym.**
   - `internal/heddleengine/store.go:376` refuses a whole store that names an unknown target.
   - Bolt's `heddle.json` travels between machines (Neve and OSL-1046) by git, and each machine may run a different lym build.
   - Counterexample: a repository entry aimed at `agents-prompt` lands. A machine whose lym predates this task then fails every heddle render and every passage read for that hub.
   - Adding the target bumps no store `version`, so the "keeps a higher version" guard from `15de027c` does not apply.
   - Fix: D7 bumps the store version when it writes `agents-prompt`, or states that every machine must be on the new build first.

6. **LOW — D9 and D8 against D1's "no exemption table".**
   - Several of D9's and D8's own bounds are items a Claude session gets and a Codex session does not:
     - CLAUDE.md files above the repository root (D9, carried by no decision).
     - Nested subdirectory CLAUDE.md files, which Claude loads when it reads files there and Codex never reads below its working directory. The record states no bound for these.
     - `@` imports that are not expanded (D8).
     - A hook that was left out (D8).
   - Either the manifest leaves these out, so it is not "one item per thing the session receives", or the parity test fails on them.
   - D9 also says nothing of a subdirectory that holds both `AGENTS.md` and `CLAUDE.md`: Codex reads only `AGENTS.md` there.

7. **LOW — D5: the promised order is undefined for some items.**
   - D5 promises Claude's order "for the same items". But Claude's appended prompt holds only the deny notice, the creel trust sentence, the subagent passage and the passages.
   - Claude gets the skill listing and the plugin contexts natively, outside that file, so D5 gives them no defined position.

8. **LOW — D8 against D10: "runs no hook" cannot match `Prepare`'s output.**
   - D10's verb shares the manifest function and runs no hook. D8 marks a hook "omitted" only after running it.
   - So the verb's manifest can never match what `Prepare` records for a failed hook.
   - The bench report D11 asks for would show "present" where the real run left the hook out.

9. **LOW — Scope Out: "lym gives no Codex session the orch role" is false under one setting.**
   - `orchSpec`'s model choice has no engine, so the orch takes shuttle.yaml's `engine:`.
   - A hub with `engine: codex` launches the orch on Codex, with a Claude model key.
   - Fix: scope it out with that bound stated, or refuse that launch.

10. **LOW — D1: the guards channel does not say which denies it covers.**
    - `internal/shuttleengine/spec.go:68-72` describes the Agent-tool, AskUserQuestion and python PreToolUse denies, which are toggleable through the claude engine's own config.
    - If they count as engine settings, D1 compares nothing for them, and a Codex autonomous seat has no counterpart deny on its question tool.

**Claims I attacked**

- D1, manifest cannot drift from the launch: broken (finding 2).
- D1, no exemption needed: broken by the bounds in D8 and D9 (finding 6).
- D2, the scan holds Claude wording to Claude blocks: broken by Go-filled markers (finding 1).
- D2, the twill spike's fork facts: held.
  - The spike ran on 0.164.0, and `spawn_agent`'s own description defaults `fork_turns` to `all`.
  - The binary's "fork_context is not supported in MultiAgentV2" string does not contradict it.
- D3, every producer passes its engine: not attacked further. Covered by finding 3 through D4's bound.
- D4, a missed producer becomes a refusal: broken (finding 3).
- D5, `developer_instructions` exists and a blank value is refused: held, from the binary string. Order: weak (finding 7).
- D6, PreToolUse covers Codex children; the default cap is three and a key raises it: held (spike report).
- D7, a shipped `claude-file` entry reaches Claude only since `15de027c`: held. Cross-version store: broken (finding 5).
- D8, global file under the shared home lock: broken (finding 4). Verb against hook output: weak (finding 8).
- D9, one file per directory and the 32 KiB cap: held. Nested files: unstated (finding 6).
- D10, the verb writes nothing: held as stated.
- D11, editing another board entry is allowed: held.
- Scope, orch out: broken under a hub setting (finding 9).
- Commit attribution: `codex_git_commit` is removed and false, and no commit carries Codex's trailer. Held.

**Further tool calls (5 used)**

1. Grep of Go files outside `claudeengine` for Claude-only wording. My context held only the stencil sweep, not Go-filled text.
2. Grep of the spike report for the fork parameter, of the code for `CLAUDE_CONFIG_DIR`, and of heddle for its unknown-target handling. These were needed to test the fork fact, D8's source and D7's store compatibility.
3. The matching lines in the Go files from call 1. I needed to tell prompt text from comments.
4. Grep of `loomengine` for AskUserQuestion, and of the Codex home's location. This confirmed the Discussion's Go-filled `mode_rules` and the shared home.
