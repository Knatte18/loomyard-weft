```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I ran each pair of decisions in the record against the other, then checked the acceptance criteria and constraints against the decisions they touch.

**Pairs checked**
- D1 × D2 (marker, outputs and kept copy keyed on review path and attempt token; stale removal before the chair starts): held.
- D1 × D2 (done marker only after the seat gate, with the kept copy written first): held.
- D1 × D2 (chair end stops the assistant, the copy only on a done marker): held.
- D1 × D6 (resume line exits the shell, so a revive dies, the probe misses the assistant, and the marker gets the outcome): held.
- D1 × D6 (a first turn that never starts fails unretried, marker `not started`): held.
- D1 × D3 (chair's wait bound against the assistant timeout and the chair's own `review_timeout_min`): broken in part, F4.
- D1 × D11 (missing assistant block, then not started and its marker, ordered after stale removal): held.
- D1 × D7 (`CreelHooks` false, pane-mail strand, assistant told to ignore `[creel]`): held.
- D2 × Added-after "report path inside the sandbox" (`/tmp` against "under TMPDIR or the worktree tree"): broken, F3.
- D2 × D6 (worktree writes reach the fixer commit, which the rejected alternative cites): held.
- D2/D3 fan × Added-after "opt-in Codex subagent fan" (drop the Open risk, close Q4): broken, F1.
- D2 × D12 (D2 promises the assistant the context the chair gets; D12 keeps the shipped entries and `nicknames` off `codex-file`): broken, F2.
- D2 × D13 (D13 supports macOS; D2's `/tmp` path is unresearched on macOS): broken, F5.
- D3 × D4 (config parses only the model-spec; the effort refusal comes at the engine, so the assistant does not start): held.
- D3 × D2 (round record against the per-round pick): held.
- D4 × D5 (`CODEX_` scrubbed from reed's env, `CODEX_HOME` set inline): held.
- D5 × D6 × Constraint "sandbox … except home rules" (automatic trust loads project-layer rules): broken, F6.
- D5 × D13 (Prepare recreates a missing link; a regular `auth.json` left after a login in the lym home): broken, F7.
- D5 × D8 (trust-prompt backstop): held.
- D5 sessions link × D7 rollout reads: held.
- D6 × D9 (skill-load turn first, prompt line returned): held.
- D6 × D10 (deny notice in the prompt file, hooks render every row): held.
- D10 × D2 (subagents run with no hook, as both state): held.
- D12 × D6 (`project_doc_fallback_filenames` against `AGENTS.override.md` shadowing): held.
- Acceptance criteria and constraints against D1–D13: covered by the pairs above, plus F1 to F7.

**Findings**

F1 — MEDIUM — consistency — decision-record.md `## Open risks` first bullet (line 207); support-log.md question ledger Q4 (line 109)
The parent's "Added after Discussion: Opt-in Codex subagent fan allowed before Step 4" (decision-record.md:168–172) says to drop that Open risk and close Q4.
Both are still there: the risk still asks the parent, and Q4 reads "Open".
The rubric says an Open-risks line that such an answer settles is a consistency finding whose fix drops the line.

F2 — MEDIUM — design — D2 (line 54) against D12 (lines 155–159)
D2 says the assistant "gets the context the chair gets (operator, 2026-10-27)".
D12 keeps both shipped heddle entries on `claude-file`, and the repository entry `nicknames` reaches Codex only after an operator `lym heddle set` (Open risks, line 208).
So by default the Codex seat gets neither the `CLAUDE.local.md` text the chair loads nor the repository entry, and D12 contradicts D2's claim.
Evidence: `lym heddle list` shows every entry on `claude-file` or `orch-prompt`; `internal/heddleengine/render.go` `fileText` renders only entries that name the target.

F3 — LOW — consistency — D2 (line 58) against Added-after "Assistant report path inside the sandbox" (line 176)
The parent decision says the report goes in the warp worktree's tree or a lym-owned per-round directory under TMPDIR.
D2 hard-codes `/tmp`, which is not under `$TMPDIR` when TMPDIR is set elsewhere.
It is still writable, since the spike lists `/tmp` and `$TMPDIR` as separate writable roots (spike report:283), but D2 departs from the recorded decision's wording without saying so.

F4 — LOW — design — D1 bound (line 47) against D3 (line 74)
The chair's wait lasts up to the assistant's start, plus `assist_reviewer_timeout_min`, plus one await cap.
The chair itself is bounded by `review_timeout_min`, 240 by default (`internal/loomengine/template.yaml:24`).
D3 sets the assistant's timeout to 60 and states no ordering between the two keys.
If an operator sets the assistant's timeout at or above the chair's, the chair times out while waiting on the marker, so the assistant gates the round, against the Goal. Nothing refuses that pair.

F5 — MEDIUM — design — D13 (line 163) against D2 (line 58) and Open risks third bullet (line 209)
D13 supports Codex seats on Linux and macOS and refuses only Windows; Scope excludes only Windows.
D2's report path rests on `/tmp` being writable, and the record leaves macOS untested (`/tmp` → `/private/tmp`, keyed by the resolved target).
That defers a fact the design rests on, on a platform D13 explicitly supports.
Either research the macOS sandbox's writable roots and record the evidence, or scope macOS out the way Windows is.

F6 — MEDIUM — design — D5 automatic trust (lines 91–96) and D6 bound (lines 108–110) against Constraint line 185
The Constraint implies only the home's rules can lift the sandbox, and says lym renders none.
Codex 0.164.0's `load_exec_policy` also loads `rules/*.rules` from each trusted Project layer's config folder (`codex-rs/core/src/exec_policy.rs`, lines 674–686 at `rust-v0.164.0`). It skips them only when `ignore_user_and_project_exec_policy_rules` is set, and the TUI cannot set it (spike report:291).
D5 trusts every launch's worktree and main checkout automatically, so a committed `.codex/rules/` allow rule would run commands outside `workspace-write`.
D6's bound ("only its prompt keeps it off target files … anywhere in the worktree") and D5's Warn, which names only `.codex/config.toml`, both understate this edge.

F7 — LOW — design — D5 (line 93) against D13 (line 165)
D13 relies on Prepare recreating `auth.json` as a link after a logout.
A login in the lym home after that logout creates a regular `auth.json` there, and `fslink`'s `prepareLink` refuses to clobber an existing path (`internal/fslink/fslink.go`, `prepareLink`).
The record doesn't say whether Prepare then fails, skips, or replaces the file, so the "missing link" path doesn't cover this state.

Tool calls:
- grep for `review_timeout_min` in `internal`: the chair's default timeout, which F4 compares against D3's assistant timeout, was not in my context.
