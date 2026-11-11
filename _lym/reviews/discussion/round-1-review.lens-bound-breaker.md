```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked each bound, guarantee, ordering promise and code claim in `decision-record.md`.

**Claims attacked**
- D7, "sessions started before this change resume and record the default directory": broken (F1).
- D2, a resume with no record finds the transcript among the listed directories and the default (D6): broken (F2).
- Scope, D1 and D4, a hub with no accounts keeps today's behaviour and the launch neither sets nor unsets `CLAUDE_CONFIG_DIR`: broken (F3).
- D3 step 1, a concurrent writer's record wins so the worktree ends with one record: held among picks, broken against a resume's record write (F4).
- D5, the merge rule means an idle session never lowers a busier session's figures: held for honest input. The verb's own input is unbounded (F5).
- D5, `&lt;Bolt&gt;/.lym/claude/usage/&lt;account&gt;.json` is written by name: broken for a name that does not match `[a-z0-9-]+` (F5).
- D3 step 4, the fallback takes the candidate whose blocking windows "all reset soonest": ambiguous, and there is no tie rule (F6).
- D3 step 2, the macOS candidate rule and the Windows claim about `.credentials.json`: unbounded and unsourced (F7).
- D2 with D5, a kept record whose name is no longer listed: the status line is silently lost (F8).
- D4, a reed relaunch replays the stored command and keeps the directory: held. It also keeps an old directory after the record changes (F9).
- D3, a new account is seeded at the lowest count over weight: held.
- D1, the tracked-layer refusal is told by configengine's provenance: held. `StackLayers` keeps the origin layer of each top-level key (`internal/configengine/layers.go:604-628`).
- D4, no caller names the variable: held. Only `claudeengine/scope.go:40` names it today.
- D6, the `env` module's value reaches panes only, not lym's own process: held. `hubgeom.go` `paneEnv` loads it only for panes.

**Findings**

F1. Severity BLOCKING, class `design`, location D2 and D7.
- Summary: whether a session started before this change can resume depends on launch order. If any fresh launch in the worktree comes before the resume, the worktree is recorded on a picked account, and the resume then runs there and is refused.
- Counterexample: the change lands. In the prime, a fresh session (the dobby, or any other fresh lym session) starts before the orch's resume. Under D2 the fresh launch has no record, so it picks account B and records it. The orch's resume then finds a record whose directory exists, keeps it, and launches under B. The orch's transcript is under `~/.claude/projects`, so the resume check refuses.
- The same happens in a task worktree that is mid-run at landing: a fresh reviewer is spawned before the writer is resumed.
- Evidence:
  - `decision-record.md:66-73`: a fresh launch with no record picks; only a resume with no record searches for the transcript.
  - `decision-record.md:179`: D7 says these sessions resume on the default directory.
  - `claudeengine/resume.go:29-38,56-58`: the check refuses when the session has no transcript under the directory it reads.

F2. Severity MEDIUM, class `design`, location D2's resume search set.
- Summary: a resume with no record searches only the listed accounts' directories and the default. That leaves out an `env` module `CLAUDE_CONFIG_DIR`, which the board says a hub can hold today. D2 cites D6, but D6's set of known directories includes the `env` value, so the two sets disagree.
- Counterexample: a hub sets `env` `CLAUDE_CONFIG_DIR=/x` today and then lists accounts. The orch resumes with no record. Its transcript is in `/x`, which is not searched, so the default is recorded. D4 then unsets `CLAUDE_CONFIG_DIR` in the launch line, and the resume is refused.
- Evidence:
  - `decision-record.md:71-73` (D2's search set) against `decision-record.md:165` (D6's known directories).
  - The board body under "Built since" says `CLAUDE_CONFIG_DIR` is allowed in the `env` module.
  - `decision-record.md:114-116`: D4's export order and its unset of the default.

F3. Severity MEDIUM, class `consistency`, location D2 and D3 against Scope, D1's criterion and D4.
- Summary: D2 and D3 never gate on a non-empty account list. Read as written, a hub with no accounts fails every fresh launch and gets records written on resume, which breaks "keeps today's behaviour".
- Counterexample, fresh launch: with `claude.accounts: []`, D2 says a fresh launch with no record picks (D3). D3 step 2 finds no candidate and fails the launch. D4 says "with no record and no accounts, the engine exports nothing", but it never says the pick is skipped.
- Counterexample, resume: with the list still empty, a resume with no record records a directory. With an `env` `/x` that is the default (F2), so D4 then unsets `CLAUDE_CONFIG_DIR` and overrides the `env` value.
- That contradicts D1's criterion that the launch neither sets nor unsets the variable, and D4's "an `env` module value stands as today".
- Evidence: `decision-record.md:26`, `:68`, `:93`, `:115`, `:231`.

F4. Severity MEDIUM, class `design`, location D2's record write on resume, against D3 step 1.
- Summary: only a pick writes the record under the hub lock with a re-check. Nothing states that a resume's record write takes that lock or refuses to overwrite an existing record.
- Counterexample: in the prime after a restart, the orch resume and a fresh dobby launch run at once. The dobby picks B under the lock and writes the record. The orch's resume, which read no record before the pick, then writes the default and overwrites B. The worktree's sessions are now split over two directories, which D2 and D7 rule out.
- Evidence: `decision-record.md:86-88` (the pick's lock and re-check) against `decision-record.md:70-73` (the resume's write, with no lock stated).

F5. Severity MEDIUM, class `design`, location D5's status line verb.
- Summary: the new verb has no stated bound on its account argument or on its input.
- Path traversal: the verb writes `&lt;Bolt&gt;/.lym/claude/usage/&lt;account&gt;.json` from an argument that no rule checks against `[a-z0-9-]+` or against the list. A name like `../../config/claude.machine` escapes the usage directory.
- Unlisted names: an unlisted name creates a stray file.
- Forged input: any agent or shell can run the verb with stdin it makes up. For example, `used_percentage: 100` with `resets_at` years ahead. Under the "later reset wins" merge, that window sticks and passes the account over until that time, and no session's honest figures can lower it.
- Evidence: `decision-record.md:131-137`, `:143`. The guard rule in the rubric (the attack-surface question) asks each new verb to state its bound.

F6. Severity LOW, class `consistency`, location D3 step 4.
- Summary: "the candidate whose blocking windows all reset soonest" has no defined ranking key when a candidate has several blocking windows, and the fallback has no tie rule.
- Counterexample: A is blocked on `five_hour` until +1h and on `seven_day` until +5d. B is blocked on `seven_day` until +2d. Ranked by the earliest blocking reset, A wins. Ranked by when every block has cleared, B wins.
- Evidence: `decision-record.md:96`; the matching criterion is at `:239`.

F7. Severity LOW, class `design`, location D3 step 2.
- Summary: on macOS every listed account is a candidate, including one that was never logged in. No bound states what that lets through. An unattended run picked onto it stalls at Claude Code's login prompt, and the fallback can choose it as well.
- Separately, the Windows half of "On Linux and Windows … `.credentials.json`" has no source: D4's facts cite Linux and macOS only.
- Evidence: `decision-record.md:90-92` against `decision-record.md:122-123`.

F8. Severity LOW, class `design`, location D2's record name against D5's condition on a listed account.
- Summary: the record stores the account name and is kept whether or not that name is still listed. D5 renders the status line only when the record names a listed account.
- Rename counterexample: the operator renames an account and keeps its directory. Every worktree on that account stops recording headroom, with no notice, and the pick reads the account as free.
- Reuse counterexample: an old name is given to a different directory. The status line then writes one account's figures under another's name.
- Evidence: `decision-record.md:61`, `:66`, `:130`.

F9. Severity LOW, class `design`, location D4's relaunch claim against D2's invariant that every session runs under the recorded directory.
- Summary: a reed relaunch replays the stored launch command, which still carries the directory from when the strand was first launched. D7 says the operator changes a worktree's account by removing the record.
- Counterexample: the operator does that. A reboot's reed Resume then relaunches the stored strands (the orch, for one) under the old directory, while the next fresh session picks and records a new one. The worktree now runs on two directories. D2 does say a resume keeps the directory, but the record no longer matches.
- Evidence:
  - `decision-record.md:118`: D4's relaunch claim.
  - `decision-record.md:218`: the operator changes an account by removing the record.
  - `internal/reedengine/lifecycle.go:184-190`: Resume relaunches non-live strands.

**Tool calls**
- One grep of `internal/reedengine` for relaunch or replay, and a read of `hubgeom.go`'s `paneEnv` (lines 125-160). The manifest did not cover reed's relaunch path or the body of `paneEnv`.
