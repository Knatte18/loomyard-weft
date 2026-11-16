```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens. Find pairs of decisions in the record that act on the same state, then check whether both decisions still hold together.

**Pairs checked**
- D1 × D4 (one spec per engine against the prompt-engine refusal): held.
- D1 × D4 (scans over production spec literals, with the shuttle CLI skipped and the hand-written mark confined to `internal/shuttlecli`): held.
- D1 × D5 (manifest read back from the `developer_instructions` launch argument): held.
- D1 × D7 (single-engine heddle targets not compared, while `agents-prompt` renders into `claude-prompt` and `codex-prompt`): **broken**, F1.
- D1 × D8 (the run record marks a plugin context omitted; tier0 fakes the hook runner): held.
- D1 × D10 (shared probe and hook runner; plugin contexts compared by key alone): held.
- D1 × D11 (the bench report opens with the run's own manifest record, which lives in the run directory): **broken**, F2.
- D1 × D12 (the nudge and the question deny as compared channels): held.
- D2 × D3 (the term scan and the engine-block copy check against the per-engine edit directive): held.
- D2 × D6 (`fork_turns` wording against the explicit `reasoning_effort` on a fresh spawn): held.
- D2 × D7 (Claude-only claude-file text in heddle Go literals): held, since the allowlist bound admits it.
- D2 × D12 (Codex nudge lines are Go literals outside `claudeengine`, so the scan forces Codex wording): held.
- D3 × D4 (the edit directive's engine and the prompt engine come from the same model choice): held.
- D4 × D10 (the verb renders each engine's prompt for its own engine): held.
- D5 × D6 (the subagent passage comes first in the developer instructions): held.
- D5 × D8 × D9 (ordering of the developer-instruction items): held.
- D5 × D10 under D8 (the 128 KiB cap measured with hooks unrun): **broken**, F4.
- D5 × D12 (deny notice placement): held.
- D6 × D12 (child calls keyed per chain through the child's `transcript_path`): held.
- D7 × D9 (heddle codex-file renders `AGENTS.override.md` carrying `AGENTS.md` and `CLAUDE.md`, with imports expanded): held. heddle owns both, and its ReRender list includes `CLAUDE.md` (`internal/heddleengine/target.go:65`).
- D10 × D12 (Prepare writes the nudge state into the temporary run directory): held.
- D12 × existing PostToolUse setting (the nudge joins the one setting): held.
- D12 × existing PreToolUse setting (the question deny against the guard entries): **broken**, F3.
- Acceptance criteria and Constraints against D1–D12: each criterion matches the decision it names. The Constraints' seam placement (rollout reading in `codexengine`, the lookup check in a shared package) agrees with D12.

**Findings**

**F1 — MEDIUM, consistency.** Location: `decision-record.md` D1, para 1, against D7.
- D1 keys an item from `claude-prompt` or `codex-prompt` to one engine and does not compare it.
- D7's `agents-prompt` renders through exactly those two targets. The record never says `agents-prompt` items are keyed as shared and compared.
- So if Codex drops an `agents-prompt` delivery, the parity test still passes, and the Goal's "a gap is a bug, and a test fails on it" fails for that channel.
- Fix: state in D1 that an item from `agents-prompt` (and `agents-file`) is keyed by its shared origin and compared.
- Evidence: decision-record.md lines 32 and 122. `internal/heddleengine/doc.go:12` shows the shared target has no file of its own and renders into both.

**F2 — MEDIUM, design.** Location: `decision-record.md` D11 against D1.
- D11 says a bench report opens with the run's own manifest record. D1 puts that record in the run directory.
- Shuttle deletes the run directory when a run ends done, unless KeepPane is set. A real producer run, which D11 requires, does not set KeepPane.
- So the record D11 makes the report open with is gone once the bench run succeeds.
- Fix: write the record somewhere that outlives the run directory, or name how the bench keeps it.
- Evidence: `internal/shuttleengine/wait.go:2033-2041` (`cleaned := outcome == OutcomeDone && !run.spec.KeepPane` … `os.RemoveAll(run.runDir)`). `docs/research/codex/engine-trial/codex-engine-trial-hooks-guards-skills-2026-11-03.md:10` says a done run otherwise removes its run directory.

**F3 — LOW, consistency.** Location: `decision-record.md` D12, para 1.
- D12 says the nudge joins the one `hooks.PostToolUse` setting. It says nothing like that for the question deny's `PreToolUse` entry.
- The guard rows already ride one `hooks.PreToolUse` setting. A second `-c` of the same key replaces the first, so a separate deny setting would wipe the guards, or the guards would wipe the deny.
- Fix: state that the deny entry joins the guards' one `PreToolUse` setting.
- Evidence: `internal/shuttleengine/codexengine/command.go:234,245`.

**F4 — LOW, design.** Location: `decision-record.md` D10 against D5 and D8.
- D10's hook runner lists every plugin hook unrun, so the developer instructions it measures carry no session-start output.
- D5's 128 KiB refusal therefore cannot show in the verb's output. The verb can pass a seat whose real launch Prepare refuses, and the D11 report would show both engines fine.
- Fix: state in D10's bound that its size check leaves out the plugin contexts.
- Evidence: decision-record.md lines 99–101 and 165–169.

**Tool calls**
1. Grepped `internal/heddleengine` for the file targets' render files and ReRender lists. The context lacked how `agents-file`, `codex-file` and `AGENTS.override.md` are rendered, which the D7 × D9 and D1 × D7 checks needed.
2. Grepped `internal/shuttleengine` for run-directory removal. The context lacked whether a done run keeps its run directory, which the D1 × D11 check needed.
