# Support log — codex-same-as-claude

## Interview

Autonomous run: no operator answered, so each turn below is a self-made pick, recorded in the question ledger.

### Turn 1 — exploration

- Board: the entry's body, the project `codex-engine-trial`, notes `codex-support-levels` and `codex-assist-review-trial`, and task `codex-writer-bench-osl`.
  No `Decision for codex-same-as-claude:` note exists.
  Peers: `weaver` (Discussion-Write) and `adopt-build-revision` (Darn-Write); neither touches launches or stencils.
- Five lookup subagents mapped: launch assembly per engine, heddle passages per engine, Claude-only stencil wording, Codex CLI 0.164.0 facts, and model routing per seat.
- Gaps found beyond the entry's list:
  - Every producer but the assist seats renders the Claude-worded edit directive, and both engine copies are thinner than the shared one.
  - Codex's standing passages ride in the prompt file, not a developer channel.
  - The subagent passage and subagent effort are Claude-only, and Codex caps concurrent children at three.
  - Shipped `claude-file` entries reach Claude only since `15de027c`; this worktree's `AGENTS.override.md` holds only the nicknames.
  - The user's global file, the plugin skill listing and plugin session-start hooks never reach Codex.
    The scribe plugin's session-start hook, which tells Claude to load `scribe:prose` and `scribe:conversation`, is one such context.
  - Codex stops adding instruction files at 32 KiB.
- Facts settled from the Codex binary and the twill spike report: `developer_instructions`, `default_subagent_reasoning_effort`, `fork_turns`, guard hooks covering children, the thread cap key.
  `codex_git_commit` is "removed, false" in `codex features list`, and no commit in the history carries Codex's trailer, so `lym-no-attribution` needs no Codex counterpart.

### Turn 2 — design picks

- Parity is checked over a launch manifest of keyed items per channel, not over text (D1).
- A prompt engine on the spec turns a missed producer into a refusal (D4).
- Standing items go to `developer_instructions` (D5).
- User setup flows through a `claudeengine` seam (D8).
- The bench rule goes on the board (D11).

### Turn 3 — lens review

Six forks, one per lens; every fork returned a report.
While checking the findings, the writer found two more gaps, now D12: Claude's batching-nudge hook and its AskUserQuestion deny have no Codex counterpart.
The Codex launch command is a shell string composed through `internal/shell` (`internal/shuttleengine/codexengine/command.go`);
the developer instructions reach `codex` as one argument under Linux's 128 KiB per-argument cap, which D5 bounds.

- leanness, D2 hit list: fixed, list and its plan note cut.
- leanness, open risk on launch length: fixed, D5 bounds the one argument the developer instructions ride as; the hook-environment risk was cut as a restatement of D8's bound.
- leanness, auto-mode assumptions restate decisions: fixed, reduced to pointers with the reasoning.
- leanness, machine facts in D8 and D9 bounds: fixed, those bounds are gone (imports expanded, files above the root carried).
- leanness, scribe example in D8: fixed, moved to turn 1.
- leanness, D2 history sentence: fixed, cut.
- leanness, `default_subagent_model` fact: fixed, cut.
- leanness, Scope "In" duplicates the decisions: fixed, one line.
- leanness, D2 term list undefined: fixed, the closed list is named.
- board-scope, parity per role, not per seat: fixed, D1's test runs every seat in `seatengine`'s tables and every other producer of a launch spec.
- board-scope, `codex-writer-bench-osl` left contradicting itself: fixed, D11 lifts its replay bounds and says its route is redesigned before it reruns.
- board-scope, the assist trial's seat changes mid-trial: kept as an open risk for the parent, since only the parent schedules those runs.
- interactions, single-engine heddle targets against "no exemption": fixed, D1 keys them to their engine and does not compare them.
- interactions, the shared edit directive's fate: fixed, D3 removes it, moves the guard's exemption, and Constraints name the background change.
- interactions, hook identity against output: fixed, the manifest keys contexts by plugin and hook, the run record marks delivery, and D10 says so.
- interactions, shared-home `AGENTS.md` against per-worktree Claude config: fixed, D8 sends the text per session in developer instructions.
- interactions, D8 and D9 carve-outs hiding gaps: fixed, imports are expanded, files above the root carried, and nested files named.
- interactions, order undefined: fixed, D5 states the order.
- interactions, cap ignores the global file: fixed by D8's move; the global file is no project document.
- interactions, tier0 against hook runs: fixed, D1's test fakes the probe and the hook runner.
- existing-paths, shipped-entry change never reaches live worktrees: fixed, D7 has the hub reconcile render file targets.
- existing-paths, the scan reaches orch stencils: fixed, D2 skips the orch places.
- existing-paths, a locally edited seeded directive: fixed, D3 states the bound.
- existing-paths, `agents-prompt` reach and `show`: fixed, D7 states both.
- walkthrough, one spec for both engines meets D4's refusal: fixed, D1 builds one spec per engine with the prompt rendered for it.
- walkthrough, the verb shows a nominal role: fixed, D10 takes a seat and renders its prompts, and D11's report opens with the run's own record.
- walkthrough, bench-osl has no runnable route: fixed with board-scope's finding.
- walkthrough, Codex config keys without their table: fixed, D6 names the `agents` table with the evidence.
- walkthrough, order undefined: fixed with interactions' finding.
- walkthrough, an omitted hook passes parity: fixed, an omitted item counts as missing in the record's verdict.
- walkthrough, cap ignores the global file: fixed with interactions' finding.
- bound-breaker, Go-filled Claude wording escapes the scan: fixed, D2 covers Go-filled text and scans production Go literals.
- bound-breaker, the manifest cannot see a dropped delivery: fixed, D1 reads the manifest back from the launch.
- bound-breaker, a forgotten producer passes as hand-written: fixed, D4 refuses an unset engine except from `lym shuttle run`, and a scan checks spec literals.
- bound-breaker, the shared home's global file races across worktrees: fixed with interactions' finding.
- bound-breaker, `agents-prompt` breaks an older lym: fixed as a stated bound in D7; no shipped entry uses the target.
- bound-breaker, carve-outs and nested files: fixed with interactions' finding; D9 names nested files and files beside an `AGENTS.md`.
- bound-breaker, order undefined: fixed with interactions' finding.
- bound-breaker, the verb cannot match a failed hook: fixed with interactions' finding.
- bound-breaker, the orch launches on Codex under a Codex default engine: fixed, D4 pins the orch spec to Claude.
- bound-breaker, the spec-level denies: fixed, D12 gives Codex the question-tool deny.

## Rejected alternatives

- A golden comparison of the two engines' rendered prompt text: the engines word their copies differently by design, so text equality would fail on every engine block.
- A manifest computed from the launch inputs: it would still list an item whose delivery `Prepare` dropped.
- An exemption table for content items with no Codex counterpart: every gap found has a Codex channel, so the table would be empty and invite silent waivers.
- Keeping the standing passages as a prefix of the prompt file: they stay in the user turn, below the authority Claude's system prompt gives them.
- Writing the user's global file into `AGENTS.md` in lym's Codex home: every hub on the machine shares that home, while a Claude config directory can differ per worktree, so launches would overwrite each other's copy.
- Linking plugin skills into Codex's native skill directories: the documented ones are the operator's home or the worktree's `.agents/skills`, which lym does not own or would have to seed as a new heddle target, and the listing in the developer instructions needs neither.
- Running plugin session-start hooks as Codex `SessionStart` hooks: whether Codex turns such a hook's output into model context is not established, while running them at `Prepare` is deterministic.
- Copying `@` imports as written with a warning: Codex would then lack text Claude reads.
- Mapping the Claude subagent model to a Codex model: no such mapping is configured, and Codex's tool text says to leave `model` unset.
- Refusing every seat with an engine-specific need on Codex: Codex has a counterpart for every need found.
- Moving the batching nudge's settings to a shared module: that changes a key's meaning and needs a config reinterpretation, while a codex.yaml block needs none.
- A `lym heddle render` way forward for shipped-entry changes in place of a reconcile render: every live worktree would keep stale agent files until someone ran it.
- Scoping the orch role in: its model is a Claude model key in orch config, and its text is the fixed embedded role.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): how is parity tested? A manifest read back from each engine's launch, per seat in `seatengine`'s tables and per other producer of a launch spec, Codex ⊇ Claude over compared channels, at `tier0` on fixtures.
- Q2 (auto-pick): which text is swept? Every stencil and spec under `contracts/` and production Go literals, bar the orch places; held by a scan with a closed term list.
- Q3 (auto-pick): where do Codex's standing items go? `developer_instructions`, Claude's appended-prompt order first.
- Q4 (auto-pick): the edit directive: one full copy per engine, replacing the shared stencil and the thin assist copies.
- Q5 (auto-pick): the user's Claude setup: per session in developer instructions, imports expanded; neither the operator's global file nor this repository holds an import today, and no CLAUDE.md sits above the repository root on this machine.
- Q6 (auto-pick): plugin listing and session-start hooks: from Claude's session scope, rendered into developer instructions; hooks run at `Prepare` with a timeout.
- Q7 (auto-pick): subagents: `agents`-table effort default and explicit `reasoning_effort` on fresh spawns; thread cap from a codex.yaml key, default 16.
- Q8 (auto-pick): shipped `claude-file` entries move to `agents-file`, a shared `agents-prompt` target is added, and the hub reconcile renders file targets.
- Q9 (auto-pick): the bench rule goes into the `codex-engine-trial` project's Rules and a section on `codex-writer-bench-osl`, whose replay route is that bench's own design.
- Q10 (auto-pick): the orch role is pinned to Claude; `lym shuttle open`, Codex resume and lym's read-back features stay out.
- Q11 (resolved by research): do shell guards reach Codex subagents? Yes, `PreToolUse` fires for children (twill spike, 0.164.0).
- Q12 (resolved by research): does Codex add a commit trailer under lym? No: `codex_git_commit` removed and false in 0.164.0, no such trailer in history.
- Q13 (auto-pick): Claude's batching nudge and question deny reach Codex through `PostToolUse` and `PreToolUse`, with the nudge's settings in codex.yaml.
- Q14 (open, for the parent): should `codex-assist-review-trial`'s three runs all run before this lands, or all after?
