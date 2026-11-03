```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on a concrete case and checked the end state against the decision's stated purpose.

**Findings**

1. **MEDIUM — D5, trust key.** The record adds the launch's worktree root to the trust list, but Codex keys trust on the repository's main checkout.
   - Case: a Codex seat launched in this worktree.
   - Its `.git` is a file that reads `gitdir: …/loomyard/.git/worktrees/codex-engine`, so it is a linked worktree of `loomyard`.
   - The hook-delivery report, case 5, shows Codex in a linked worktree saying "Trusting will apply to the repository root" and writing the entry for the main checkout.
   - So an entry keyed on the worktree path likely never matches, and the trust prompt shows at every launch. The startup classifier then accepts it, and Codex writes a main-checkout entry.
   - D5's next render drops that entry, because lym's record lacks it. The prompt therefore comes back at every launch, and the post-startup check warns each time.
   - Purpose ("started Codex with no prompt") is missed.
   - Fix: record the main checkout root, resolved from the worktree's `.git` file without running git, which keeps PATTERN-fabric-git. Or record both roots, or keep any entry Codex added when re-rendering.

2. **MEDIUM — D1/D2, the chair can read the report before it is final.** The chair's `lym shuttle await <report> <marker>` returns as soon as the report file exists. That can happen before the assistant has finished it.
   - Case: the assistant writes a malformed report and ends its turn. Its review-parse gate (`ReviewGateEntry`, `PassOnCap`) re-prompts it to rewrite the file.
   - The chair's await has already returned `ready` on the first draft. The chair consolidates the malformed or partial file while the assistant is still rewriting it.
   - The same happens if the assistant writes the file in more than one step.
   - Purpose ("the chair consolidates the assistant's report") is met only by chance.
   - Fix: wait on a done marker that seatengine writes when the assistant's run ends done with its gate passed, beside the ended marker, instead of on the report itself.

3. **MEDIUM — D6, untested launch prompt.** No report tested a positional `[PROMPT]` on the Codex launch line. The spike typed every prompt into the composer, and the hook-delivery report does not say how its prompts went in.
   - Untested: whether the positional prompt submits on startup, whether it survives a folder-trust prompt shown first, and whether `SessionStart`/`UserPromptSubmit` then fire. The start rule depends on those hook lines.
   - The rubric's "deferred research question" bullet covers this: the design rests on a fact nobody has checked.
   - Fix: record the evidence, from Codex docs or a one-off check. Or fall back to delivering the first turn through the tested paste path.

4. **LOW — D3, empty entries vs validation.** D3 says an empty entry runs no assistant, and also that every entry is parsed as a model-spec.
   - Case: the default `""`, or `["astra[medium]", ""]`.
   - The existing `ValidateModelSpecList` refuses an entry that does not parse as a model-spec, so the D3 lists fail through that path.
   - Fix: state that empty entries are exempt, and that the default empty value reads as "no assistant" rather than as an invalid entry.

5. **LOW — D11/D2, await-forks in a fanned assistant round.** In a fanned round the assistant renders the fan rules, and `burler-cluster-fan` has the handler wait with `lym shuttle await-forks`. That verb fails on an engine without `OutstandingForkReader`, which codexengine lacks under the record's Scope.
   - D11 lists only "spawn and inheritance and how a file is written" as engine blocks.
   - Fix: name the wait and the lens-report saving as part of the engine block too. On Codex, the wait is the in-turn wait on its own subagents.

**Per decision**
- D1: assistant times out → marker written → chair's await returns → goes on. Met, apart from finding 2.
- D2: fanned discussion round with an Astra assistant. Met in principle, apart from findings 2 and 5. The `.lym` path sits inside the workspace (anchor equals the worktree root), and the copy after the chair ends holds.
- D3: default `""` → no seat. Met in intent; the validation wording conflicts (finding 4).
- D4: `luna[low]` → codex engine, effort `low` allowed. Met.
- D5: launch in a linked worktree. Not met (finding 1).
- D6: a launch with skills. Met only if the positional prompt works (finding 3). The start rule itself holds, because a session start plus a turn start lets startup pass while the pane reads busy.
- D7: `Stop` payload → turn end, `last_assistant_message` as the message; `turn_aborted` → interrupt. Met.
- D8: a send through paste with a separate Enter. Met.
- D9: `scribe:prose` → reads `…/1.1.0/skills/prose/SKILL.md`; classified from the rollout through the Stop payload's `transcript_path`. Met.
- D10: `python3 -V` → the Bash `PreToolUse` hook denies it with the tested output shape. Met.
- D11: Claude seat → unchanged bytes; Codex seat → engine blocks. Met apart from finding 5.
- D12: no `AGENTS.md`, `CLAUDE.md` present → the override opens with `CLAUDE.md`. Met.
- D13: Linux symlink to `auth.json`. Met.

**Tool calls**
- One Bash call read the worktree's `.git`, the seat `Timeout` fields and `ReviewGateEntry`. My context lacked whether lym worktrees are linked worktrees, whether a per-seat timeout exists for advisors, and the gate's `PassOnCap` behaviour.
