```yaml
lens: walkthrough
outcome: reported
```

**Decisions walked (lens: walkthrough)**

- D1 — assisted round retried within the same round: **broken** (chair starts first; stale marker removed only before the assistant starts, so the chair can read the prior attempt's `done` marker)
- D2 — solo Codex assistant in this worktree writing its report under the anchor's `.lym`: **broken** (path outside the sandbox's writable roots; report never written)
- D3 — `discussion_assist_reviewer_model: ["", "sol[medium]"]`, rounds 1-3: held (round 1 none, rounds 2-3 sol, fan solo)
- D4 — `astra[xhigh]` with default `efforts`: held (refused naming `efforts`)
- D5 — first launch in this warp worktree: held for the main checkout root (`.git` → `/home/knatte/Code/loomyard-LYMHUB/loomyard/.git/worktrees/codex-engine`); the `[tui]` part of the render has no keys to render (partly)
- D6 — launch with skills, then a reboot: held (skill-load first turn; resume exits shell, run ends died)
- D7 — turn ended by `Stop`, then an Esc interrupt: held
- D8 — startup on trust prompt, paste send: held
- D9 — `scribe:prose` under the plugin cache: held (resolves under a version dir, tail match from rollout)
- D10 — `sed -i` via Codex Bash: held (PreToolUse deny shape matches spike)
- D11 — assistant engine block missing for `codex`: held (assistant not started, chair runs)
- D12 — this repository (no `CLAUDE.md`, no `AGENTS.md`, repo instructions only in `CLAUDE.local.md`): **broken** (Codex seat gets no repository instructions)
- D13 — `CreateFileLink` for `auth.json` on Linux: held

**Findings**

1. **BLOCKING · design · D2** — The assistant's report path under the anchor's `.lym` is not writable by a `workspace-write` Codex session, so the assistant never delivers a report.
Walk: Codex's workspace is the warp worktree (`codex-engine/`). The worktree's `.lym` is a symlink to `codex-engine-weft/.lym`, where `.lym-anchor` lives, so the anchor's `.lym` is outside the workspace. The spike records writable roots as "the workspace, `/tmp` and `$TMPDIR`" and that Codex resolves symlinks to their real path ("The worktree's `.scratch/` is a symlink, so Codex reported and keyed everything by the resolved path"). The write fails; the review-parse gate re-prompts until its budget, the run ends at `assist_reviewer_timeout_min`, the marker reads `timeout`, and the chair goes on without it — on every round. D2's purpose is missed. The support log already applies this fact to `_lym` (Turn 2: "`_lym` is a junction out of the worktree, so a Codex report written there falls outside the sandbox"); `.lym` is the same kind of link.
Evidence: `ls -la` worktree root (`.lym -> /home/knatte/Code/loomyard-LYMHUB/codex-engine-weft/.lym`); `codex-engine-weft/.lym-anchor`; `docs/research/codex-spike-2026-10-24/report.md:10,283`; `_lym/discussion/decision-record.md:55`; `_lym/discussion/support-log.md:24`.

2. **MEDIUM · design · D12 (and D6)** — The `CLAUDE.md` fallback gives the Codex assistant none of the repository instructions the chair gets in this repository.
Walk: the repository has no `CLAUDE.md` and no `AGENTS.md`; the chair's instructions come from heddle's `claude-file` target, rendered as `CLAUDE.local.md`. Every heddle entry targets `claude-file` only (`lym-batching`, `lym-plan-edits-before-gate`, `nicknames`), so `codex-file` has no entries. D12's render finds no `AGENTS.md`, no `CLAUDE.md` and opens with nothing; D6's `project_doc_fallback_filenames=["CLAUDE.md"]` finds no file either. End state: an assistant with no repository or lym instructions — misses D12's stated purpose ("the assistant must get the repository instructions the chair gets") and the operator's 2026-10-27 parity requirement. Two of the entries are heddle's shipped layer, which is code this task can change, so the Open risk calling it "an operator edit through `lym heddle set`, not code" is wrong for them.
Evidence: `ls` (`CLAUDE.md`, `AGENTS.md`, `AGENTS.override.md` absent; `CLAUDE.local.md` present); `lym heddle list` output; `internal/heddleengine/render.go:255-270`; `internal/heddleengine/target.go:37-38`; `_lym/discussion/decision-record.md:149,183`.

3. **MEDIUM · design · D5** — The render step is handed no data for "the `[tui]` settings Codex otherwise writes on its first start": the record never names the keys, so the implementer must find them out. The twill spike records two: `screen_reader_detection_done` and a `model_availability_nux` table; neither is in the record or its evidence.
Evidence: `_lym/discussion/decision-record.md:87,186`; `docs/research/codex-twill-spike-2026-10-24/report.md:17`.

4. **LOW · design · D1/D2** — A retry of the same round can hand the chair the previous attempt's `done` marker and report.
Walk: marker and report paths are stable per round (D1, D2); archiving before a retry copies only the `.assist-<seat>` copy, the `.lym` original and its marker stay. seatengine starts the chair first, then other seats; D1 removes stale files only "before it starts the assistant". A chair whose first `lym shuttle await <marker>` precedes the assistant's start (slow start, trust-prompt handling, start retry) gets `ready: true` on the stale `done` marker and reads last attempt's report. Removing them at table build, before the chair starts, closes it.
Evidence: `internal/seatengine/doc.go:51`; `_lym/discussion/decision-record.md:38-39,59`; `internal/shuttlecli/await.go:49`.

Tool calls:
- One Bash call listing `CLAUDE.md`/`AGENTS.md`/`AGENTS.override.md`/`CLAUDE.local.md`, printing `.git`, and grepping `render.go`/`target.go` for the codex-file render — context lacked the repository's instruction files, the main-checkout pointer and the codex-file render body.
