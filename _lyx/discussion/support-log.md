# Support log — bugfix-sessions

## Interview

Autonomous mode: no operator answered; every pick below is the Discussion producer's own, recorded in the ledger.

1. Scope. Read the board entry and the related notes `orch-resume-without-attach`, `orch-notice-channel`, `operator-shortcuts` and `skills-in-one-turn`. Scope is the entry's bug sections plus the terminal half of `orch-resume-without-attach`; the resume verb, notice channels and shortcuts stay on their notes.
2. #395 delivery. Read `sendVerified`: it accepts a send once the text appears in the pane, which an unsubmitted input box also satisfies, so the check passes on exactly the reported failure. Picked submission confirmation through a provider input-box reading, plus pacing text and Enter as two steps in `claudeengine`.
3. #399. `reflectHalt` reflects on every halt; picked skipping the spawned reflection for `escalation-to-human` only, keeping the note.
4. #400. The attach probe already matches runs by output-file set; picked extending its respawn branch to remove terminal predecessors' live strands. Webster's recovery strands are handled in Webster, since they bypass the probe.
5. Crash-resume. The voucher mechanism should already cover a completed step; the reported case shows a gap whose cause needs reproduction. Picked fixing the observable outcome and leaving the cause to the plan writer with candidate causes, plus evidence in the note.
6. #390. Picked "no turn end between boundary and trigger" as the freshness rule, and skipping the skills step after a compaction.
7. #393. A web check found Claude Code's `/reload-plugins`, which rebuilds the model-facing skill list but not the slash-command parser; since `skills-in-one-turn` loads skills through the Skill tool, that suffices. Picked typing it before every reload.
8. Restart race. The watcher already handles SIGINT/SIGTERM through a cancel context; picked a stopping record in state and a bounded wait in `start`.
9. #386. Burler rounds already run through a shuttle gate; picked adding a review-parse gate entry so the session repairs its own file, and one self-check verb under `lyx burler`, since the gate-parity pattern requires it.
10. Attach. Operator decision 2026-10-06 settles it; picked keeping `--no-attach` as a hidden no-op since batten passes it.
11. Testing. The board asks for a regression test per fix through real CLI wiring; picked the CLI entry per fix with substrate fakes only.

## Rejected alternatives

- #395, bracketed paste in `reedengine.SendText` (`load-buffer` + `paste-buffer -p`): Claude Code collapses a large paste into a `[Pasted text]` placeholder, which would defeat the needle check in `sendVerified` and trigger a replay; it also changes reed's provider-free transport for a provider quirk. Pacing in `claudeengine` plus submission confirmation covers the race without either cost.
- #395, confirming submission through a `UserPromptSubmit` hook in the events file: a long-lived session launched before the change lacks the hook, so the orch session would never confirm; the input-box reading works for every live session.
- #399, suppressing the halt note as well: breaks `PATTERN-friction-capture` and the terminal Friction-Reflect row's input.
- #400, reattaching to the halted session instead of removing it: the halted run reached a terminal outcome and its turn ended; reviving it mixes the old attempt's context into the re-run the operator asked for.
- #400, removing every strand of the same role: would kill legitimate parallel same-role strands (numbered `-2`, `-3`) with other outputs.
- Crash-resume, dropping the crash-resume trigger for `step` entries: hides a real driver death between steps.
- #390, a time-based freshness window for compactions: arbitrary, and wrong on a busy or idle session alike; turn order is the real signal.
- #393, restarting the Claude Code process on every cycle: costs the session id and Remote Control link in compact mode and a large resume's startup time, for what `/reload-plugins` does in place.
- #393, only reporting the unknown skill: `skills-in-one-turn` already logs it; the session still could not use the skill.
- #393, `update-plugins.sh` signalling the orch: a new cross-module channel, a feature by the board's rule.
- Restart race, a new restart verb: forbidden by the board's no-new-verbs rule.
- #386, a tolerant frontmatter reader: hides malformed review data and weakens the sole parser; the session can repair its file.
- #386, only restating the quoting rule in the stencil: the error already states it and the run still failed; the round needs a repair turn.
- Attach, removing `--no-attach` outright: batten and existing callers pass it and would break.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): fix for #395? Submission confirmation via a `claudeengine` input-box reading, plus paced text then Enter (D1, D2). Rationale: covers every delivery path and every cause of a lost Enter.
- Q2 (auto-pick): how many extra Enters? Two, then fail. Rationale: bounds blind key presses while covering one more lost Enter after a retry.
- Q3 (auto-pick): which halts skip reflection? Only `escalation-to-human` (D3). Rationale: the issue names that kind; other kinds carry lyx defects worth reflecting on.
- Q4 (auto-pick): remove or reattach a halted producer's strand? Remove, matched by output-file set (D4). Rationale: bounded by the existing attach-probe identity.
- Q5 (auto-pick): Webster recovery strand rule? Remove on every terminal outcome with a live strand and on `webster reset` (D5).
- Q6 (auto-pick): crash-resume cause? Left to the plan writer's reproduction, with candidate causes; the outcome and the note's evidence are fixed here (D6, D7). Rationale: Step 2's bound forbids deep code research at Discussion.
- Q7 (auto-pick): freshness rule for a compaction? No turn end between boundary and trigger (D8).
- Q8 (auto-pick): skills after a compaction? Not retyped; only after `/clear` (D9).
- Q9 (auto-pick): #393 fix? `/reload-plugins` before every reload (D10). Open risk R1 if the command misbehaves.
- Q10 (auto-pick): restart race? Stopping record plus bounded wait in `start` (D11).
- Q11 (auto-pick): #386? A review-parse gate entry reusing `ParseReview`, plus the self-check verb `PATTERN-gate-self-check-parity` requires beside every mechanical gate (D12). Rationale: the pattern wins over the board's no-new-verbs line.
- Q12 (auto-pick): `--no-attach`? Hidden accepted no-op (D13).
- Q13 (auto-pick): test entry per fix? The CLI wiring named in D14, substrate fakes only.
