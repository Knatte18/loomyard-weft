---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: decision
    location: _lyx/plan/10-status-seams-spawn-detached-push.md:78-101
    summary: The plan leaves PATTERN.md 211 bytes over the headroom line and makes card 10 stop and ask ly:orch, so the declared PATTERN changes cannot land inside the plan.
    fix: "In card 10, replace the paragraphs from 'That cut leaves 211 bytes owed' to 'it makes no PATTERN.md edit until the answer gives room for all 371 bytes' with a complete cut. Extend the numbered list of rewritten entries, each given by entry name and new wording after its em dash, until the cuts total at least 363 bytes (measure each entry with wc -c, bytes not characters, since the em dash is three bytes), so that PATTERN.md after card 15 is at most 15564 bytes. Every extra cut keeps its entry's rule, (test) marker and background link. Sources not yet examined: the introductory sentence on PATTERN.md line 3 ('The structural invariants of the loomyard code, ...' becomes 'Structural invariants, one line per entry: when it applies and what it requires.', 25 bytes) and the roughly eighty entries the 17-entry list does not touch. Name the cut in card 10, which runs before card 15's growth, and remove the stop-and-ask instruction. If no further wording cut exists, the plan must say so in card 10 by naming the operator decision it needs (which entries to merge or remove, or moving HeadroomBytes) as a plan-time open question for ly:orch to answer before the run starts, not as a stop in the middle of Webster's batch."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/08-push-lock-pending-flags.md:38-39,46,58,61
    summary: A holder's respawn from a pending flag bounds the side path and the working directory but trusts the flag's slugs and its other-side path, and no test row reaches either.
    fix: "In card 8, make step 4 build the respawned PushSpawn from the flag's validated side path and slugs only, never copying the flag's opposite-side path field (a pending.code flag keeps only WarpPath, a pending.records flag only WeftPath, the other emptied), and drop with a Warn naming the flag any flag whose PauseSlug or TeardownSlug holds a path separator or a dot-dot element, since TeardownLockPath builds a file path from the teardown slug. State this in the 'What bounds it' paragraph. Add two rows to TestCoalesceFlagged_PendingFlags: a pending.code flag file that also carries a WeftPath respawns a request with an empty WeftPath, and a flag whose TeardownSlug is a dot-dot path is dropped with a Warn and spawns nothing."
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/15-command-engine-replaces-switched-rows.md:66-67,89
    summary: TestCommandEntry names a non-string switch row but no row for a non-string command, though the card prescribes a per-key wrapped refusal for command too.
    fix: "In card 15's TestCommandEntry bullet, add a row for a non-string command (command: 3) beside the non-string switch row, with the same assertion that the message carries the row, the key command and its way forward clause."
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/15-command-engine-replaces-switched-rows.md:72,89-90
    summary: The Command row of minimalEntries and the valid builds in TestCommandEntry run over newTestEnv, which leaves Env.Setting nil, so settingRef refuses them with its Env.Setting is nil error.
    fix: "In card 15, say that the minimalEntries Command row's env func and TestCommandEntry's building rows set env.Setting to fakeSetting answering the switch and command references, leaving ExecutableSetting nil where the plan says it still builds. Without it TestEveryEntry_RejectsUnknownConfigKey fails on the Command row's valid build."
---

### [BLOCKING] PATTERN.md ends 211 bytes over the headroom line and card 10 stops

**Location:** `_lyx/plan/10-status-seams-spawn-detached-push.md:78-101`

**Issue:** The focus asks whether the plan leaves `PATTERN.md` within `HeadroomBytes` and names the cut in a card that runs before the growth.
I measured the arithmetic.
The three declared entries grow the file by 70 (push-both-sides), 112 (recipe-setting-lookup without "told") and 189 (command-row, line plus newline) bytes, 371 in all.
`PATTERN.md` is 15556 bytes at the merge base and `HeadroomBytes` is 15564, so 363 bytes must be cut.
The 17 rewrites card 10 names save 152 bytes; I recomputed each entry and the total matches.
Card 10 then states 211 bytes are owed, that "no further wording cut keeps every rule", and that it stops and asks `ly:orch`.
That leaves the plan unable to land its own declared PATTERN changes: card 15's growth fails the headroom lint, and card 10's own PATTERN edit is withheld until an answer arrives mid-run.
The claim that no further cut exists is unsupported: the named list touches 17 of about a hundred entries, and the file's introductory sentence on line 3 is untouched.
A plan that schedules a stop for a decision it could settle in-plan, or must have settled before the run, is a sequencing defect.

This re-raises round 2's F2, which that round's fixer deferred as an operator decision because "every rule-preserving wording cut I found adds up to 152 bytes".
That evidence does not hold as a bound: it is the fixer's own search result, the plan shows no search of the other entries, and the file's introductory sentence on line 3 is not an entry and carries no rule, yet it is outside the list.
I do not claim 211 bytes of further wording cuts exist; I claim the plan has not shown they do not.
Whichever way that is answered, the plan owns the answer: the choice between a further cut and an operator decision must be made before the run, not by a stop at card 10.
I classed it `decision` for that reason, since only the plan author or `ly:orch` can make it and no fixer can settle it by editing wording alone.

### [MEDIUM] A pending-flag respawn trusts the flag's slugs and opposite-side path

**Location:** `_lyx/plan/08-push-lock-pending-flags.md:38-39,46,58,61`

**Issue:** The focus asks whether the plan bounds everything a holder's respawn trusts from a flag file.
The side path is bounded: a flag for a side the holder does not push is respawned only when that path belongs to the lock directory's worktree.
The working directory is bounded: the respawn sets `WorkDir` to the board directory and ignores the stored value, and the first `TestCoalesceFlagged_PendingFlags` row set reaches it with a flag holding another one.
Two inputs remain unbounded.
The flag holds a whole `PushSpawn`, and the bound examines only the one path of the side it is named for, so a `pending.code` file that also carries a `WeftPath` respawns a child pushing that path.
The pause and teardown slugs are respawned as read; the card lists them as let through and bounds them only by "only `markPending` writes them".
The teardown slug becomes a path through `TeardownLockPath` in card 9, so a slug with a separator makes the child create a lock file outside `HubScratchDir`.
The writer is the local user, so the severity is MEDIUM and not BLOCKING, but the card states its bound as complete and no test row reaches either input.

### [LOW] No test row for a non-string command in TestCommandEntry

**Location:** `_lyx/plan/15-command-engine-replaces-switched-rows.md:66-67,89`

**Issue:** The card wraps the typed-accessor refusals of `switch`, `command` and `timeout_s` separately, each with the row, the key and a way forward clause, and `TestCommandEntry` reaches `switch: 3`, a missing `command`, a dotless reference, `timeout_s: 0`, a non-int `timeout_s` and an unknown key.
A non-string `command` goes through its own wrap with its own key in the message and no row reaches it.
This is the focus's "non-string value for `switch` or `command`" case; every other refusal the card prescribes has a named row and ends in a way forward clause.

### [LOW] The Command row of minimalEntries builds over an Env with no Setting

**Location:** `_lyx/plan/15-command-engine-replaces-switched-rows.md:72,89-90`

**Issue:** `settingRef` refuses a named reference when `env.Setting` is nil, and `newTestEnv` in `internal/shedrecipe/fixture_test.go` leaves `Setting` nil.
The card says `minimalEntries` gains a `Command` row with a valid config and that its valid build is pinned by `TestEveryEntry_RejectsUnknownConfigKey`, and that `TestCommandEntry` builds a producer over `newTestEnv`, without saying either sets `Setting`.
Run as written, the valid build refuses and the unknown-key test fails.

## Rejected

- Nil-seam tests: with `Command.Setting` and `Command.ExecutableSetting` added to `envkit.nilLegal`, `FullEnv` filling `ExecutableSetting` and the `Command` fields `TaskDir`, `LogPath`, `Run`, `Lyx`, `Slug`, `Dir`, `Rerun`, `walkNil` reports nothing, so `TestFullEnv_FillsEverySeam` stays green.
  Batten's `wire` fills `ExecutableSetting`, `Command.TaskDir`, `Command.LogPath` and `Command.Run`, so `TestWire_EverySeamFilled` needs no new entry; loom's `intentionallyNil` gains exactly those four paths and drops the six switched ones.
  `Forced` is a `[]string`, which `walkNil` skips, and `GOOS` is a string.
  Not a defect.
- Entry line cap: the written `PATTERN-recipe-setting-lookup` line is 316 characters, `PATTERN-command-row` 186 and `PATTERN-push-both-sides` 214, all under `MaxEntryLineChars` (320), so `go test ./internal/pattern` passes on line length after the card that writes each.
  The declared `PATTERN-recipe-setting-lookup` text, written verbatim, is 321 characters, one over the cap, so the declared text and the repo's `internal/pattern` test conflict.
  Card 15 drops the one word "told", cites the declared text and records the departure; no admissible wording keeps both, so I record the conflict here and raise no finding against the plan.
- Whether card 10's cut is accurate: the 17 rewrites save 152 bytes as stated; the defect is the unpaid remainder in F1, not the arithmetic.

