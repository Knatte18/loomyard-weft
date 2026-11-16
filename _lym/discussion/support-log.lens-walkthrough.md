```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case and checked the end state against the decision's stated purpose.

**Findings**

1. **MEDIUM. D1 and D4 conflict, and so the parity test cannot check the prompt or edit-directive channels.**
   - Case: the parity fixture for the darn-writer role, Prepared on both engines "for the same spec".
   - Under D4, the spec carries the engine its prompt was rendered for, and Codex `Prepare` refuses a spec whose prompt engine is `claude`.
   - So the shared fixture spec can only work with an empty prompt engine. Both manifests then say "hand-written", and the prompt-engine and edit-directive items match trivially.
   - The test never shows that a production prompt rendered for Codex carries Codex blocks. That is the gap the bench hit, and D1 says "a gap is a bug, and a test fails on it".
   - Fix: compare per-engine specs that are the same except for their prompt engine. Each engine's prompt is rendered for that engine, for example through `seatengine.Prompts`.

2. **MEDIUM. D10's verb shows a hypothetical session for a role, not what a run's session got. That falls short of D11's purpose.**
   - Case: `lym shuttle manifest` for the darn writer's role in a bench worktree.
   - The verb has no task, so no prompt is rendered. The prompt-engine and edit-directive items are missing, and those are the channels the bench got wrong.
   - The skill loads come from the role, but the darn writer's real skills come from its seat: the role skills plus the hub's language skills, set in seatengine.
   - Plugin contexts appear by name only, because the verb runs no hook.
   - So a report that "opens with what each engine's session got" shows a nominal pair, not the run's actual launch. Meanwhile D1 already writes the real Codex manifest into the run directory, and D11 does not point to it.
   - Fix: D11's report should open with the run's own manifest record for the Codex side. The verb should be used only for the side that did not run, and it should render the producer's prompt for the seat when one is named.

3. **MEDIUM. D11 marks `codex-writer-bench-osl`'s steps as superseded, but leaves the entry with no route it can actually run.**
   - Case: the bench reruns as a replay of `824eb2b6`.
   - The real darn producer runs a board task against the current parent branch.
   - "Only the model swapped through `run_settings`" does not say how the replay starts from the parent of `824eb2b6` with the old body.
   - After D11 the entry's steps 1–3 are superseded and nothing runnable replaces them, so a rerun "this way" cannot start as written.
   - Fix: D11 should state the replay route, or say plainly that the bench entry needs its own redesign before it reruns.

4. **LOW. D6 names Codex config keys without the table they sit in.**
   - The binary's strings `max_depth default_subagent_model default_subagent_reasoning_effort job_max_runtime_seconds` sit together in one struct's field list, beside the thread-cap key, which is under `agents`.
   - So `default_subagent_reasoning_effort` most likely belongs under the `agents` table too.
   - Set at top level, it would probably be ignored or refused, and the effort would never be applied.
   - Both key names do exist in 0.164.0: 22 hits for `default_subagent_reasoning_effort` and 33 for `max_concurrent_threads_per_session`.
   - Fix: state the table for each key, with the evidence.

5. **LOW. D5 says "in Claude's order", but that order is undefined for two of its items.**
   - Claude's appended system prompt holds the subagent passage, the deny notice, the creel trust sentence and the passages.
   - The skill listing and the plugin contexts reach Claude by other native channels and are not in that file.
   - So the order for those two items is unspecified, and the acceptance bullet "in Claude's order" cannot be checked for them.

6. **LOW. A runtime gap under D8 passes D1's parity check.**
   - Case: the scribe SessionStart hook times out at Codex `Prepare`.
   - Its manifest item is still present, only marked omitted, so a key-presence comparison sees parity.
   - The same is true for an `@` import that was copied as written.
   - D1's purpose ("a gap is a bug") holds only in the fixture test. At run time the gap shows only as a warning.
   - Fix: D1 should say that an omitted item counts as missing for the verdict that D10 and the record report.

7. **LOW. D9's size cap covers only the files on the path from the repository root to the working directory.**
   - The new `$CODEX_HOME/AGENTS.md` from D8 is not counted.
   - If Codex counts the global file against `project_doc_max_bytes`, the cap computed from the path files alone falls short by the size of the global file.
   - Fix: include the global file in the sum, which costs nothing.

**Held**
- D2, checked on the loom discussion lens review on Codex: the waiting step `lym shuttle await-forks` is a Claude fork-audit poll (`internal/shuttleengine/await.go`), and D2's mapping from waiting to `wait_agent` covers it once the block moves. Purpose met, provided the await step is part of the moved passage.
- D3, checked on a gatefixer prompt on Codex: the purpose is met once `Directive` takes the engine.
- D6, checked on a six-fork review: a cap of 16 is enough, and forks inherit the session's effort. Purpose met, apart from finding 4.
- D7, checked on a new worktree render: the shipped entries reach `AGENTS.override.md`. Purpose met.

**Tool calls**
- Searched the Codex binary's strings for the agents-table key names, to settle which table D6's keys belong to. My context had only one ambiguous line.
- Searched for the await-forks implementation and reed's launch files, to check how D2 covers waiting on Codex. My context did not show whether await-forks is Claude-only.
