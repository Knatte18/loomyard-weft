```yaml
lens: interactions
outcome: reported
```

Scope: the interactions lens over the decision record's decisions, acceptance criteria and constraints, read as an outsider.

**Pairs checked**
- D1 wait × D2 seat gate: **broken** (F2)
- D1 resume × D2 report path: **broken** (F1)
- D1 stale-marker removal × D2 fresh instruction directory: held on the fresh path (no stale marker can exist); breaks on resume (F1)
- D1 hold deferral × D1 chair inside await: held (the chair is busy, not held)
- D1 stop when the chair ends × D2 copy after the chair ends: weak (F8)
- D1 quiet hold × D7 pane-mail strand: under-specified (F7)
- D2 fanned assistant × D10 shell guards: **broken** (F3)
- D2 fanned assistant × D7 subagents fire no hooks: held for completion; the guard side is F3
- D2 report under `.lym` × D6 workspace-write: held
- D3 timeout × D1 wait bound: held (60 is below 240)
- D3 per-round list × D2 fan from the chair's round: held
- D4 effort refusal × D1 not-started marker: held; the refusal lands as a silent not-start (LOW, no finding)
- D4 `CODEX_` scrub × D6 inline `CODEX_HOME`: held
- D5 whole render × D8 trust backstop × D6 first turn: **broken** (F4)
- D5 links × D7 rollout read through payload path: held
- D6 empty resume line × D1 resume: held (seat resume attaches and never relaunches)
- D6 prompt-file notice × D9 skill-load first turn: weak (F9)
- D9 path reads × D6 sandbox and D10 rows: held (reads are allowed everywhere, and `cat` matches no row)
- D10 shell-edit row × D2 report written by apply_patch: held (apply_patch is not Bash)
- D11 validation refusal × Goal and D1 "never gates": **broken** (F5)
- D11 × `PATTERN-engine-blocks` text: **broken** (F6)
- D11 Claude blocks × D2 "no-assistant bytes unchanged": held by assertion
- D12 × D6 fallback flag: held
- D13 × D1 not-started marker: held

**Findings**

F1 **BLOCKING**, D1 × D2 (resume)
- Summary: the assistant's report path and ended marker sit in a fresh temp directory that a probe or resume can never name again.
- Evidence: D2 writes the report "under the half's instruction directory below the anchor's `.lym`" with the marker beside it. That directory is `os.MkdirTemp` per `Review` call. `ProbeReview` and `ResumeReview` build the table from `probeDirOf()`, a fixed `round-probe` path "never created". So the probe's assistant outputs never match the started run, and the seat probe cannot find it. D1's resume rule then writes the marker into the probe directory. The chair's prompt names the original temp path, so its `lym shuttle await` never sees the marker and the chair waits until its own timeout.
- Fix: put the report and marker at a path that is stable per round, for example derived from the review path and round.

F2 **MEDIUM**, D1 × D2 (gate)
- Summary: the chair's wait ends when the report file first exists, not when the assistant has finished.
- Evidence: D1's wait is `lym shuttle await <outputs> <ended marker>`, which returns as soon as any path exists. D2 puts a review-parse gate on the assistant that re-prompts it to rewrite a malformed report. So the chair can read a half-written report, or one the gate is about to replace, and consolidate it.
- Fix: have the engine also write a "finished" marker when the assistant's run ends done, and have the chair await finished-or-ended.

F3 **MEDIUM**, D2 × D10
- Summary: the fanned assistant's Codex subagents run without the shell guards, and D2's bound does not say so.
- Evidence: the Constraint "A Codex subagent fires no hook of its own" means no `PreToolUse` deny fires for them. A reviewer fork could run raw `go test ./...` past `PATTERN-gate-slot`, or python. D10 claims every row guards the session. D2's bound names only the missing fork audit.
- Fix: state that bound in D2, or run the assistant solo.

F4 **MEDIUM**, D5 × D8 × D6 (trust)
- Summary: rendering `config.toml` whole undoes the trust backstop every launch, and the trust prompt can swallow the first-turn line that startup waits on.
- Evidence:
  - Per the linked-worktree Constraint, Codex keys trust on the main checkout's root, but D5 adds the linked worktree's own root.
  - If that entry is not honoured, D8 accepts the prompt and Codex writes the main-root entry. D5's next render, made from lym's record only, drops it, so the prompt returns on every launch.
  - D5's post-startup check looks for the launch's own root, which lym wrote, so it never warns.
  - Whether D6's positional first turn survives the trust dialog is untested, yet D6 waits on `SessionStart` from that first turn.
- Fix: record the root Codex actually keys on, or carry Codex-written trust entries into lym's record.

F5 **MEDIUM**, D11 × Goal and D1
- Summary: an assistant misconfiguration fails the whole review round, so Codex can gate a run.
- Evidence: D11 says "A block missing for the seat's engine refuses the table at validation". That refusal stops the Claude chair too. The Goal says the assistant "never gates a round", and D1 says an assistant never ends the step. Other assistant failures (D4 effort, D13 Windows) correctly become not-starts.
- Fix: make a missing engine block an assistant not-start, not a table refusal.

F6 **MEDIUM**, D11 × `PATTERN-engine-blocks`
- Summary: the new PATTERN entry is broken on the day it lands.
- Evidence: the entry says "a shared stencil names no engine's tool". But D11 moves only the passages the assistant's stencils use into engine blocks. The edit directive, `seat-directive-advisor`, the twill and webster stencils and `burler-cluster-fan`'s Claude-side text still name the Agent tool and the Edit and Write tools.
- Fix: narrow the entry to stencils that a non-default engine renders, or put the rest in scope.

F7 **LOW**, D1 × D7
- Summary: D1 does not say an assistant's held turn end stays quiet.
- Evidence: D1 copies stops and the hold deferral from advisors but not "holds a turn end quietly". A held-turn notice to the parent invites mail, which D7 turns into text typed into the Codex pane. The acceptance criteria have no quiet-hold check either.

F8 **LOW**, D1 × D2 (copy)
- Summary: the copy beside the review can be a partial file.
- Evidence: D1 stops every assistant when the chair ends, possibly mid-write. D2 then copies "an existing assistant report" with no check that the assistant's run was done.

F9 **LOW**, D6 × D9
- Summary: the deny notice reaches the session after the first turn.
- Evidence: D6 puts the notice in the prompt file. With skills, that file is the deferred prompt line, so the skill-load turn runs before the notice is read. The hooks still deny; only the announcement lags.

Further tool calls: none.
