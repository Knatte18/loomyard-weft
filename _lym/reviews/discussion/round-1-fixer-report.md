# Fixer report — discussion, round 1

Every finding's premise held against `_lym/discussion/decision-record.md`, the code it cites and the Authentication docs.
All twelve are fixed; F2 and F3 are fixed differently in part.

## Changes

- F1 (BLOCKING), fixed as proposed, then narrowed by the self-check:
  D2's fresh launch with no record, or with a gone directory, first searches every known directory for the transcripts of the sessions the worktree's own lym state can resume, under D3's lock, and records the directory holding the newest one; only a worktree with none picks.
  D3 step 1 makes that search under the lock.
  D7 rests on the search, not on launch order.
  The Auto-mode assumption now says how the operator changes a worktree's account, since removing a record re-records the transcripts' directory.
  New D2 criteria cover a fresh launch with default-directory transcripts, and a reused worktree path that still picks.
- F2 (BLOCKING), fixed differently:
  D2 gains a first rule that with `claude.accounts` empty, as always outside a hub, a launch makes no pick, searches nothing and writes no record, and with no record it exports nothing.
  Not made: "keep an existing record's export only when the list is non-empty".
  It contradicts D2's own rule that "emptying the list leaves the worktrees already on an account where they are": a worktree on account B would launch under the default after the list is emptied, and its sessions' resumes would be refused.
  An existing record is still read and exported instead.
  The D1 criterion now covers the resume command and a resume writing no record.
- F3 (MEDIUM), fixed differently in part:
  D2's resume searches every known directory (D6), the `env` module's and the process's `CLAUDE_CONFIG_DIR` included, and a new D2 criterion puts the transcript under the `env` module's directory.
  Not made: recording D6's fallback directory when the transcript is found nowhere.
  The fix-way-forward lens showed that it pins a worktree whose record the operator removed to the fallback directory, never picking.
  The resume now writes no record, runs under the fallback directory, and is refused by the resume check; a D2 criterion covers it.
- F4 (MEDIUM), fixed as proposed: D2 states that every record write runs under D3's lock and re-reads the record first; D3 gains the resume-plus-fresh-launch concurrency criterion.
- F5 (MEDIUM), fixed as proposed, the bound corrected by the self-check:
  D5's verb refuses a name outside `[a-z0-9-]+` or not listed, and drops a window out of range or reset beyond its window's length.
  The bound says a forged input can raise or lower one account's eligibility in one window until the forged reset, five hours or seven days on.
  New D5 criteria cover each case.
- F6 (MEDIUM), fixed as proposed: D4 has `Claude.OpenLaunch` read and write the record and export or unset the variable in both lines; D5 says an opened session's settings carry the status line when it writes settings, which it does only with a hub board directory. D2's session list names opened sessions. New D4 criterion.
- F7 (MEDIUM), fixed as proposed, first option:
  D2 drops reed relaunch from the record readers.
  It says a changed record or a gone directory reaches a running strand only at a launch the engine builds, and that a relaunch can recreate a gone directory empty and logged out.
  It names the way forward: end the strand and start it again with the verb that started it, after a login or a record change.
  D4's bullet points to D2.
- F8 (MEDIUM), fixed as proposed: D6's report readers read every known directory, never the worktree's record; the known set adds the directories of the hub's worktree records; the bound for history on an account in no list or record is stated. New D6 criterion.
- F9 (LOW), fixed as proposed: D3 step 4 takes the latest blocking reset, earliest first, ties to the earlier entry; the rationale and criterion match.
- F10 (LOW), fixed as proposed:
  D5 resolves the account at launch by its cleaned directory against the current list and passes the entry's current name.
  Its rationale states the bound for sessions launched before a list change.
  The D5 settings criterion covers a rename, and the support log's rejected alternative is reworded to match.
- F11 (LOW), fixed as proposed:
  D3 step 2 states the macOS bound.
  D4's facts cite the Windows location, `%USERPROFILE%\.claude\.credentials.json` following `CLAUDE_CONFIG_DIR`, confirmed on the Authentication docs' "Credential management" section on 2026-11-11.
- F12 (NIT), fixed as proposed: the Ultra line moved from Open risks to the support log's Turn 1 upstream bullet.

## Self-check

- fix-claims, LOW, D5 claimed every opened session carries the status line though `OpenLaunch` writes settings only with a hub board directory: fixed as proposed, condition stated.
- fix-consistency, LOW, D3 rationale named the old fallback: fixed as proposed.
- fix-consistency, LOW, support log's rejected path-matching alternative read against D5: fixed as proposed.
- fix-consistency, LOW, support log Q7 stated the old precedence: fixed as proposed.
- fix-way-forward, MEDIUM, a resume after record removal recorded the fallback and blocked the re-pick: fixed as proposed, a resume whose transcript is found nowhere writes no record.
- fix-way-forward, MEDIUM, the relaunch bound named no way forward: fixed as proposed, the restart through the starting verb is named.
- fix-way-forward, LOW, hand-written record form unstated: fixed as proposed, a record holding only `config_dir` is named.
- fix-proposal, MEDIUM, the forged-input bound covered lowering only: fixed as proposed, both directions and each window's limit stated.
- fix-proposal, MEDIUM, a relaunch can recreate a gone directory: fixed as proposed, bound and way forward stated in D2.
- fix-proposal, MEDIUM, sessions launched before a rename pass the old name: fixed as proposed, bound stated in D5's rationale.
- fix-proposal, MEDIUM, the search caught an earlier worktree's transcripts at a reused path: fixed as proposed, the search is limited to sessions the worktree's own lym state can resume, with a criterion.

## Disputed

None.

## Deferred

None.

## Rulings

None.
