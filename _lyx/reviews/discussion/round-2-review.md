---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4 (fifth bullet) and Acceptance criteria (Trailing words row)
    summary: D4 routes a global flag by matching the exact words -v and --verbose, so the root flag's own documented -vv form and --verbose=N fall through to the last argv only, and the acceptance criteria pin only lyx -v prime
    fix: In decision-record.md D4's fifth bullet, replace "every `-v` or `--verbose` word before the first `--` goes onto each argv instead" with a rule stated over the root's persistent verbosity flag in every spelling pflag accepts for it (`-v`, `-vv` and longer runs, `--verbose`, `--verbose=<n>`), each such word before the first `--` going onto every argv, wherever it sits relative to the shortcut's name; apply the same spelling rule to the `--help`, `-h` and `--json` interception (`--json=<value>` included). In Acceptance criteria, extend the row starting "Trailing words reach the last argv" with `lyx -vv prime` running both `orch start` and `reed attach` with `-vv`, and `lyx prime -v` running both with `-v`.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D2 (Decision and Rationale)
    summary: D2's Rationale says its held readings match shuttle's own waiting reading, but that reading also counts a live held-turn wait marker, which D2 excludes without saying why the exclusion is safe
    fix: In decision-record.md D2's Rationale, replace "The held readings, `asking` and `idle-stalled`, match shuttle's own waiting reading" with a sentence saying the held readings are the session-state half of shuttle's waiting reading, and add one sentence giving why the held-turn wait marker is left out, namely that a held turn end is a turn that ended without the run's output files, which the session reads as `idle-stalled`, so the marker adds no case the reading misses; if that premise does not hold for an advisor, add the marker to D2's held set and to the acceptance row on the hold-deferral predicate instead.
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D3 (Bound line)
    summary: D3's bound says reed's name repair pastes behind reed's activity check and that every caller is lyx's own code, but name repair pastes anyway when the activity reading errors, and shuttle's send carries text any invoker of lyx shuttle send supplies
    fix: In decision-record.md D3's Bound line, replace "every caller is lyx's own code, shuttle's sends behind its pre-write guard and reed's name repair behind reed's activity check" with a statement that the two paste paths are shuttle's send, whose single-line, non-empty text any invoker of `lyx shuttle send` may supply and which runs behind shuttle's pre-write guard, and reed's name repair, whose text is the fixed rename line and which pastes after the session-idle read and the activity check, or past an unreadable activity reading; keep the clause that size is unbounded by design.
---

### [MEDIUM] D4's verbosity routing matches two exact words, not the flag's spellings

**Location:** `_lyx/discussion/decision-record.md` D4, fifth bullet; Acceptance criteria, the row starting "Trailing words reach the last argv".

**Issue:** The focus asks which argv each forwarded word reaches.
D4 answers for the exact words `-v` and `--verbose`: each goes onto every argv, and every other word goes onto the last argv.
The root's only global flag besides help and JSON is a count flag, declared in `cmd/lyx/main.go:121` with the usage `increase log verbosity (-v info, -vv debug)`.
Its documented debug form `-vv` is one word that is neither `-v` nor `--verbose`, so under D4's rule `lyx -vv prime` runs `orch start` without verbosity and `reed attach` with `-vv`, the opposite of what D4's own stated aim ("a global verbosity flag typed before the shortcut's name ... reaches every expanded command") wants.
`--verbose=2` falls the same way, and `--json=true` escapes the help interception the same way, reaching the last argv as a local flag, which D4 says can happen only after `--`.
The acceptance criteria pin only `lyx -v prime`, so a build honouring D4 literally passes them.
The rule also applies to a `-v` typed after the shortcut's name (`lyx prime -v`), since D4's words include both sides of the name, but no acceptance row pins that placement.

### [LOW] D2 claims to match shuttle's waiting reading while dropping one of its readings unexplained

**Location:** `_lyx/discussion/decision-record.md` D2, Decision and Rationale.

**Issue:** `internal/shuttleengine/agentwait.go` counts a run as waiting when its held-turn wait marker is live or its session reads `asking` or `idle-stalled`.
D2's held set names `asking`, `idle-stalled`, `idle-done` and `dead`, and states "the advisor's held-turn wait marker is not read", but neither the Decision nor the Rationale says why an advisor with a live marker whose session reads neither held reading is safe to keep counting.
The Rationale's "match shuttle's own waiting reading" is then not accurate as written.
I found no case where the omission defers the notice wrongly: a held turn end is a turn that ended without the run's output files, which `internal/shuttleengine/sessionstate.go` names `idle-stalled`.
The gap is the missing reason, which a plan writer needs to keep from re-adding the marker read or from treating the two sets as identical.

### [LOW] D3's bound misstates where the pasted text comes from and how name repair is gated

**Location:** `_lyx/discussion/decision-record.md` D3, Bound line.

**Issue:** The structural half of the bound holds: with the text as stdin bytes of `load-buffer -`, no text can add, end or alter a command in the list, so removing the trailing-`;` escape and the `--` guard lets nothing new through on that axis.
The paste path's two callers are `internal/shuttleengine/send.go`'s `pasteVerified` and `internal/reedengine/namerepair.go`'s session name repair.
The trust half is inexact on both.
Name repair logs a Warn and pastes anyway when `operatorActiveLocked` errors ("not holding the session rename"), so it is not strictly behind the activity check.
Shuttle's send carries text supplied by whoever runs `lyx shuttle send <guid> <text>`, an agent or the operator, limited by `validateSendText` to one non-empty line; lyx's code is the caller, not the text's author.
With size now unbounded, the bound should name who authors the text that can now be larger, not only which Go code calls the transport.

## D1 verdict

The D1 directive's verdict is not a defect: `overviewShows` hides a `done` run, so the covered states D1 must rule on are `running`, `paused`, `blocked`, `awaiting` and `failed`, plus the overview's own `starting` cell (`overviewStarting` in `internal/battencli/overview.go`), and D1 and its acceptance rows give each a rule; no state name there falls outside `internal/shedengine/status.go`'s literals except `starting`, which D1 names as what the run shows rather than as a batten state.
