```yaml
lens: bound-breaker
outcome: reported
```

**Claims attacked (lens: bound-breaker)**

- D2: report and marker under the anchor's `.lym` "so the workspace sandbox covers them" — **broken**
- D1: chair reads an assistant's report only after a done marker / stale marker removed before the assistant starts — **broken** (race with chair-first start)
- D1 bound: chair's wait lasts at most the assistant's timeout plus one await cap; "always ends" — **broken** (start retries; Go process death)
- D1: probe finds a live assistant by its stable outputs — **broken** (leftover report from a prior attempt of the same round)
- D6: a first turn that never starts leaves the run not started, "which shuttle retries as any failed start" — **broken** for a dropped prompt with Codex alive
- D6 bound: only the prompt keeps the session off target files; hooks deny regardless — held for shell commands; does not cover repo-committed Codex hooks run unreviewed by `--dangerously-bypass-hook-trust` (see 6)
- D5 bound: a Codex write racing a render is overwritten; classifier accepts the trust prompt as backstop — held (a dropped root comes back at the next render; backstop accepts the prompt meanwhile)
- D5: pruning vanished roots under the lock; two concurrent Prepares lose no root — held
- D5: trust keyed on main checkout via `internal/dotgit` — held (one sample, stated)
- D7 bound: missing `Stop` ends at timeout — held
- D9 bound: missed tail match costs one retry turn — held (stated)
- D10 bound: Codex subagents' commands meet no hook — held (stated)
- D11: missing chair block refuses table; missing assistant block only not-starts it — held
- D13: Prepare passes Windows refusal before writing anything — held
- Constraint: lym renders no rules file into its home, so no operator allow rule widens the sandbox — held (`CODEX_HOME` replaces `~/.codex`)

**Findings**

1. BLOCKING · design · D2, "The assistant's report and end marker sit at a path under the anchor's `.lym` … so the workspace sandbox covers them"
Counterexample: in this very worktree `.lym` is a symlink to `/home/knatte/Code/loomyard-LYMHUB/codex-engine-weft/.lym`, and the anchor (`.lym-anchor`) sits in `codex-engine-weft/`, outside the warp worktree that is Codex's workspace. The spike found `workspace-write` allows writes only to the workspace, `/tmp` and `$TMPDIR`, and that Codex keys everything by a symlink's resolved path; a report written there falls outside the sandbox, exactly as the support log says of `_lym`. The assistant can never write its report, so it never ends done.
Evidence: `ls -la` worktree root (`.lym -> …/codex-engine-weft/.lym`); `docs/research/codex-spike-2026-10-24/report.md:10,283`; `_lym/discussion/support-log.md:24`.

2. MEDIUM · design · D1 (stale-marker removal ordering)
D1 removes a stale marker "before it starts the assistant", but seatengine starts the chair first and the assistant after it. Marker and report paths are stable per round and D2 archives a *copy* before a retry, so the `.lym` originals stay. On a retry of the same round a chair that reaches `lym shuttle await <marker>` before the assistant has started meets the previous attempt's `done` marker and reads the old report. A Codex start can be slow (trust-prompt classification, up to `start_retries` relaunches at 5 s). Fix direction: remove before the chair starts.
Evidence: `internal/seatengine/doc.go:51`; `internal/shuttleengine/doc.go:533`; decision record D1/D2.

3. MEDIUM · design · D1 (probe by stable outputs)
Attempt 1 of a round ends done, leaving its report at the stable `.lym` path. Attempt 2 crashes after the chair starts but before the assistant's start removed stale outputs; Resume's probe then "finds" an assistant by the leftover report — shuttle's attach treats all-outputs-exist as finished (`soleFinishedCandidate`/`allOutputFilesExist`), so the probe marks attempt 1's report done for attempt 2. The record clears stale outputs only on a fresh start, never on probe.
Evidence: `pattern/PATTERN-shuttle-provider-seam.md` (Completion signal); decision record D1.

4. MEDIUM · design · D1 bound ("the chair's wait lasts at most the assistant's own timeout plus one await cap"; "one wait that … always ends")
The marker is written only by the Go engine. If the half's lym process dies while the chair's reed pane lives on, nothing writes it; the chair re-runs await on `ready: false` every turn until a resume's probe, which may be never, spending a turn per keep-warm cap. Separately, the assistant's timeout starts only after start succeeds, so up to `start_retries` extra launches lengthen the wait beyond the stated bound.
Evidence: `internal/shuttleengine/doc.go:533`; `internal/shuttlecli/await.go:1-2,48-52`.

5. MEDIUM · design · D6 ("a first turn that never starts leaves the run not started, which shuttle retries as any failed start")
Shuttle retries a start only when the provider "died before it took its first input and whose strand was removed". Take a first launch where D5's render failed so the trust prompt appears and D8's backstop accepts it: whether Codex still submits the positional prompt after that accept is untested (the spike launched without one; hook-delivery used TUI/exec only). If dropped, Codex sits alive at its ready composer — not died, so no retry, and no `SessionStart` ever fires; the run waits out its timeout instead of retrying.
Evidence: `internal/shuttleengine/doc.go:533`; `docs/research/codex-spike-2026-10-24/report.md:345`; decision record D6.

6. MEDIUM · design · D6 bound (`--dangerously-bypass-hook-trust` with D5's automatic trust)
Hook-delivery case 5 ran a hook committed in the repository's `.codex/config.toml` with no prompt, once the folder was trusted and only the bypass flag was given; the spike's prompt says such hooks "can run outside the sandbox". D5 trusts every worktree root and main checkout automatically, so D6's bound lets any committed `.codex/` hook or setting run unreviewed, outside the sandbox — the record states neither this edge nor its bound.
Evidence: `docs/research/codex-hook-delivery-2026-10-28/report.md:224-250`; `docs/research/codex-spike-2026-10-24/report.md:66-82`.

Tool calls:
- Grep of `internal/shuttleengine` for start retries and `internal/reedengine` for done-when: context lacked when shuttle retries a start, which the D6 claim leans on.
