---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:75
    summary: >-
      D2 fresh-launch search names three session-id sources, but the worktree holds other resumable session ids whose records outlive the shuttle run directories, so the pre-change migration claim is false
    fix: >-
      In D2's fresh-launch bullet (the one that starts "A fresh launch with no record"), replace the closed list "the session ids the shuttle run records, the reed strands and the orch's state record" by a rule, "every record under the worktree's anchor that stores a provider session id its owner later resumes, each read through its owner module's exported reader", and keep the orch state record as one named example. Name these owners as evidence that the closed list is incomplete, by module and concept only, no signatures - the sampler row's record, the PR-Review session record, the gate fixer's record, webster's state (the master session and the batch and fixer sessions) and treadle's state - each of which keeps a session id that its owner resumes after the shuttle run directory is swept. Leave the exact owner list to Plan's quarry lookups. Keep the sentence saying the wiring passes the collected ids to Prepare and OpenLaunch as told data. Add one acceptance bullet under D2, next to the orch-state one, for a fresh launch in a worktree with no record whose only resumable session id sits in a PR-Review or sampler record, its transcript in the default directory and its shuttle run directory gone, which records the default directory and makes no pick.
  - id: F2
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:89
    summary: >-
      The resume check's live-holder guard reads the session registry, and the record never says which directory's registry it reads, so under a non-default account the guard stays on the default registry and stops catching a live holder
    fix: >-
      In D2's last bullet (the one that starts "Shuttle runs the resume check before Prepare") and in D6's first bullet, state that the check's live-holder test reads the session registry of the directory the check resolved, the same directory it stats the transcript under, and that with the no-record search it reads the registry of the directory where the transcript was found. State the bound, that the guard is unchanged for the default directory and moves with the resolved directory otherwise, so two processes never drive one session on any account. Add an acceptance bullet under D6, beside the resume-check one: a resume of a session registered as held by a live process only in a listed non-default account's registry is refused, and a resume of one whose registry entry is dead passes.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:224
    summary: >-
      D6 looks a session id up in every known directory's registry but gives no rule for combining the answers, and a known directory with no sessions directory reads unproven and can hide a dead answer from another directory
    fix: >-
      In D6's second bullet, state the combining rule for the id-only liveness read: the entries of every known directory's registry are read as one set; a known directory with no registry directory contributes no entry and is not doubt; an existing registry directory or entry that cannot be read is doubt; the answer then follows the rule one registry already follows - any live entry reads alive, dead needs a dead entry and no doubt, anything else reads unproven. Add an acceptance bullet under D6 next to the liveness one: a session registered dead in a listed account's registry, with the default directory holding no registry directory, reads dead, and one registered alive in any known directory reads alive.
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lym/discussion/decision-record.md:87
    summary: >-
      D2 refuses a resume over a gone recorded directory with a message that names the directory and the way forward, but the empty-list case states a plain transcript stat, and a fresh launch with an empty list exports the gone directory
    fix: >-
      In D2 make one rule for a recorded directory that no longer exists, whatever claude.accounts holds. State that a resume launch is refused as the second-to-last resume bullet says, with the directory and the way forward, also when claude.accounts is empty, and change the last bullet's empty-list sentence so it stats under the record's directory only when that directory exists. For a fresh launch with an empty list and a record whose directory is gone, state the outcome (a refusal with the same way forward, or the launch exports nothing and records nothing) so the launch no longer has Claude Code create the directory empty and logged out. Extend the acceptance bullet that covers the gone directory (the one starting "D2: a fresh launch with a record whose directory is gone") with a row for the empty list.
  - id: F5
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:130
    summary: >-
      D3 raises an account whose count over weight is below the others' lowest but leaves a re-added or lowered-weight account above it untouched, so that account is passed over until the others catch up and D3's claim that it takes its share from the next pick is false
    fix: >-
      In D3's counter paragraph (from "The counter file also holds each name's weight") replace the one-way raise by a set. An account that was not listed at the last pick, or whose weight differs from the stored one, has its count set to the lowest count over weight among the other listed accounts that were listed at the last pick with an unchanged weight, times its own weight, whether that raises or lowers it. State the case where no such account exists, so that a hub's first pick and a list that replaced every account at once are both defined: the count is then 0 for every account that the rule would set, and say whether a kept count of a re-added name is overwritten. Drop the sentence that counts for names no longer listed are kept if the set makes it moot. Keep the sentence that an account listed throughout under one weight is never changed.
  - id: F6
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md:327
    summary: >-
      The D3 criterion says a re-added or reweighted account never takes two picks in a row while another eligible account's share is behind, which is false for weights such as 3 and 1 and has no testable meaning for share is behind
    fix: >-
      Reword the criterion in terms of the counter: after a re-added account's count is set, or a reweighted one's, it is picked only while its count over weight is no higher than that of every other eligible account, and an account that gained weight takes no pick before its count over weight reaches the lowest of the others. Remove never takes two picks in a row, which weights 3 and 1 legitimately violate.
---

### [BLOCKING] D2's fresh-launch search names three sources of resumable session ids and the worktree holds more

**Location:** `_lym/discussion/decision-record.md:75`
**Issue:** D2 says the search covers "the session ids the shuttle run records, the reed strands and the orch's state record under the worktree hold", and D2 and D7 then rest on the claim that a session started before the change has its transcripts in the directory it ran under, so the search records that directory.
The code holds more owners of resumable ids.
`internal/loomshed/samplersession.go` resumes `rec.SessionID` from the sampler row's record, `internal/landingshed/prsession.go` resumes the PR-Review record's session id, and `internal/gatefixer/record.go`, `internal/websterengine/state.go` (`MasterSessionID` and the batch and fixer ids) and `internal/treadleengine/state.go` each keep a session id.
`internal/shuttleengine/run.go` `sweepOrphans` removes a run directory once its strand is gone, so the shuttle run record does not cover them after the sweep, and the reed strand list is live-only.
A pre-change worktree whose first launch after the change is a fresh one, with only such a record naming its sessions, finds no transcripts, picks an account, and the later resume of the session its owner holds is refused for a missing transcript.
The record's claim for pre-change sessions is therefore false, and the first-launch migration is the case that the lens review of turn 3 settled.
The fix is a rule with a stated category, not a longer list, so the enumeration stays with Plan.

### [BLOCKING] The resume check's live-holder guard reads the default registry

**Location:** `_lym/discussion/decision-record.md:89`
**Issue:** `internal/shuttleengine/claudeengine/resume.go` `CheckResume` stats the transcript, then reads the registry under `~/.claude/sessions` to refuse a resume while a live process holds the session.
D2 and D6 say the check resolves its directory by D2's resume rules and "stats the transcript", and D6's resume-check acceptance row says the files are found in that directory.
Neither names the registry.
If the registry read stays on the default directory, a session live under a second account is never seen as held, so a second process can drive it.
That is a guard that stays in the code and stops working for every non-default account, and the record introduces the second directory without stating what the guard can now let through or what bounds it.

### [MEDIUM] D6's id-only liveness read has no rule for combining directories

**Location:** `_lym/discussion/decision-record.md:224`
**Issue:** `claudeengine` `processLiveness` returns unproven when `readRegistry` fails, and a directory without a `sessions` directory fails it.
Across the known directories, which include the default, the `env` module's and every worktree record's, most have no entry for a given id and some have no registry directory at all.
`internal/shuttleengine/wait.go` accepts the "process is gone behind a live pane" proof only from a dead answer that an earlier alive read backs, so one absent registry directory that reads unproven would hide it.
D6 gives the lookup, "one match is the session's", and no combining rule.

### [MEDIUM] A gone recorded directory is handled differently under an empty list

**Location:** `_lym/discussion/decision-record.md:87`
**Issue:** The bullet at line 87 refuses a resume over a gone directory and names the directory and the way forward, as `PATTERN-refusal-way-forward` requires.
The last bullet of D2 says that with `claude.accounts` empty the check "stats the transcript under a record's directory", which for a gone directory yields the engine's plain no-transcript refusal with no way forward.
The empty-list bullet at line 68 also exports a record's directory with no existence check, so a fresh launch exports a gone directory and Claude Code creates it empty and logged out, the failure D2 describes for a relaunch.
The acceptance bullet for the gone directory is written only for a list that names accounts.

### [MEDIUM] D3's counter does not keep its no-catch-up and even-spacing claims for every case

**Location:** `_lym/discussion/decision-record.md:130`
**Issue:** The rule raises an account that was not listed at the last pick, or whose weight rose, only when its count over weight is below the lowest of the others.
A re-added account whose stored count over weight is above that lowest, and an account whose weight was lowered, are not changed, so each is passed over until the others catch up, which is the opposite of the claim at line 133 that a new, re-added or reweighted account "takes its share from the next pick".
The first-pick sentence says every listed account's count "starts at 0" when no other listed account was listed at the last pick, which also covers a list that replaced every account at once and does not say whether it overwrites a kept count.
Reweighting an account from 2 to 1 with a count of 10 shows the starvation, since its count over weight becomes 10 against others near 5.

### [LOW] The D3 re-added and reweighted criterion is not true of the counter

**Location:** `_lym/discussion/decision-record.md:327`
**Issue:** With weights 3 and 1 the counter picks the weight-3 account three times in a row after a tie, so "never takes two picks in a row while another eligible account's share is behind" is false for a reweighted account, and "share is behind" names no quantity a test can compute.
A criterion stated by count over weight, as the fix gives, is checkable and is true of the rule once F5 is fixed.

## Rejected

- D4's bound on an exported `CLAUDE_CONFIG_DIR` reaching reed's server and later panes is adequate.
  `internal/reedengine/lifecycle.go` applies the scrub to `os.Environ()` at the server spawn, `internal/shuttleengines/shuttleengines.go` `EnvScrub` reaches it from the engine's `ScrubbedEnv`, and `internal/reedengine/spawn.go` chains the `env` module's exports before the launch command, so the order D4 states holds.
  D4's acceptance row covers the server.
- Scope's and D6's cost-reporting wording is correct.
  `internal/reportengine/cost.go` reads the usage records a run keeps and reads no transcripts, and only `internal/shedadapters/sessionusage.go` and `tools/tokencount/reads.go` read transcripts through `LocateSessions`.
- D6's route for the accounts, the `env` module and the worktree-record directories is stated.
  The record gives the engine `Board` and `Anchor` in its `Dirs`, a told anchor list its caller resolves, and a `RecordReaders` that takes the `Dirs`, which answers the empty `configengine.Dirs{}` that `RecordReaders` builds today and keeps `PATTERN-cwd-resolution`'s told-geometry rule; the gap that remains is F3.
- The orch's own state record is a correct source for the fresh-launch search: `internal/orchengine/state.go` `State.SessionID` is the id `internal/orchcli/start.go` resumes.
  The finding F1 is about the owners beside it.
