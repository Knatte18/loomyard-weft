```yaml
lens: interactions
outcome: reported
```

**Pairs checked (lens: interactions)**

- D1 × D2 (stable marker/outputs per round + copy/archive before retry): broken (F1)
- D2 × D6 (report path under anchor's `.lym` + `workspace-write` writable roots): broken (F2)
- D1 × D7 ("no channel to it exists" + `CreelHooks` false → pane-mail strand): broken (F3)
- D6 × D12 (`CLAUDE.md` fallback flag + codex-file falls back to `CLAUDE.md`) with the hub's heddle state: broken (F4)
- D1 × D11 (Table.Validate refuses missing assistant directive include vs. missing engine block only not-starts the assistant): held — one is a seat-directive include, the other an engine block
- D1 × D3 (chair wait bound + `assist_reviewer_timeout_min`): held
- D2 × D3 (round record + per-round pick of model/fan): held — probe reads the record, not the keys
- D1 × D6 (Resume attaches a live assistant + resume line exits the shell): held — attaching a live pane needs no resume line; a revived pane dies and is marked died
- D6 × D7 (first turn on the launch line + `SessionStart` fires with the first prompt): held
- D6 × D9 (skill-load as first turn + deny notice in prompt file): held, ordering stated as D6's bound
- D2 × D10 (fan subagents + hook-only guards): held, both state the gap
- D5 × D8 (rendered `config.toml` + trust-prompt backstop writing into it): held — the record keeps the roots, a render restores the entry
- D5 × D13 (Prepare writing under the lock + Windows refusal before any write): held
- D3 × D4 (config validation of model keys + engine refusing an unlisted effort): held — an effort outside the list passes config and fails at Prepare, so the assistant does not start and the round goes on
- D2 × D11 (bench reading directions as engine blocks + Claude seat prompt unchanged): held
- Acceptance bullets for D1, D2, D3, D11, D12 vs. their Decisions: wording matches; D12's bullet inherits F4's premise
- Constraint `PATTERN-agent-messaging` ("assistant sends no message") × D7: held for sending; receiving covered by F3

**Findings**

F1 — MEDIUM — design — D1 × D2: a retry can serve the chair a stale `done` marker.
D2 keeps the marker and report stable for the round, and only the copy beside the review joins the archive before a retry; the `.lym` original and its marker stay. D1 removes a stale marker "before it starts the assistant", and seatengine starts the chair first, then the seats after it. On a retry of the same round the chair's `lym shuttle await <marker>` can return ready on the previous attempt's `done` marker and read the old report before the engine clears it. The removal must happen before the chair starts (at table build).
Evidence: `_lym/discussion/decision-record.md:38-39,59`; `internal/seatengine/doc.go:51`; `internal/shuttlecli/await.go:49` (returns as soon as the path exists).

F2 — BLOCKING — design — D2 × D6: the assistant's report path is outside the sandbox.
D2 puts the report "under the anchor's `.lym` … so the workspace sandbox covers them"; D6 launches `-s workspace-write`, whose writable roots are the workspace, `/tmp` and `$TMPDIR`. The worktree's `.lym` is a symlink to `codex-engine-weft/.lym`, and `.lym-anchor` itself sits in `codex-engine-weft`; Codex resolves symlinked paths (spike: `.scratch/` keyed by resolved path). The report path is outside the writable roots — the same fact the support log states for `_lym`. The assistant can never write its report, so every assisted round ends without one.
Evidence: `_lym/discussion/decision-record.md:55,97`; `docs/research/codex-spike-2026-10-24/report.md:10,283`; `ls -la` worktree (`.lym -> …/codex-engine-weft/.lym`); `_lym/discussion/support-log.md:24`.

F3 — LOW — consistency — D1 × D7: the assistant does have a mail channel.
D1 says the assistant "gets no … mail … and no channel to it exists"; D7 sets `CreelHooks` false "so creel treats a Codex strand as a pane-mail strand", and creel types notices and reminders into every such strand's pane. A post addressed to the assistant's strand reaches its Codex pane mid-review; neither Decision states that edge or its bound.
Evidence: `_lym/discussion/decision-record.md:36,111`; `internal/creelengine/deliver.go:34-35,242-246`.

F4 — MEDIUM — design — D6 × D12: Codex gets no repository instructions.
D6's `project_doc_fallback_filenames=["CLAUDE.md"]` and D12's fallback both load `CLAUDE.md`, but the repository has no `CLAUDE.md` (support log Turn 2, confirmed by `ls`). The chair's repository instructions come from heddle's `claude-file` target (`CLAUDE.local.md`), and every heddle entry, shipped and repository, targets `claude-file` alone. So both fallbacks load nothing for this repository and the Codex assistant gets none of the instructions the chair gets — D12's stated purpose ("the assistant must get the repository instructions the chair gets") and the operator's context-parity rule fail. Open risks treats it as a "may" and an operator-only edit, but the shipped entries are lym code in scope.
Evidence: `_lym/discussion/decision-record.md:97,148-149,183`; `_lym/discussion/support-log.md:23`; `lym heddle list` output (all entries `targets:["claude-file"]` or `orch-prompt`).

Tool calls:
- `ls`/`git ls-files` for `CLAUDE.md`, `AGENTS.md`, `CLAUDE.local.md` — to check whether the repository has a `CLAUDE.md` for D6/D12's fallback; context only had the support log's claim.
