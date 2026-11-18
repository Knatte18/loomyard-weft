```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness over the account/relaunch diff. List: function or branch, the case I ran, correct or not.

**Run list**

- `claudeengine.checkResumable`, `sessionHolder` and `processLiveness`: a known config directory with no `sessions/` directory. **Incorrect** (F1).
- `shuttleengine.relaunchSpec` and `relaunchUnderAccount`: a resume check that returns a warning. **Incorrect** through F1, because the warning refuses the relaunch.
- `claudeengine.validateAccounts`: `config_dir: ~/x` with an unresolvable home. **Incorrect** (F2).
- `stopForRelaunch`: `EndStrandProcess` fails and the strand is live; `Status` fails; `ReleaseStrandHold` fails. **Mostly correct.** The release-failure case is F3.
- `relaunchUnderAccount`, the failure of each step after the mark (`relaunchSpec`, `Prepare`, record write, `RelaunchStrand`, `awaitStartup`). **Correct.** Every path ends in `endFailedRelaunch` or `awaitStartup`'s own teardown, and a held strand is removable.
- `endStoppedRelaunch`, handle stop versus `StopStrand`. **Correct.** It matches the arbiter ruling: finalize once after `Run.Stop`, identity with `OutcomeDied` after `StopStrand`.
- `settleStaleRelaunch`, past bound with a dead or held strand, a live strand, and a status read failure. **Correct.** The offsets are not re-read, but the attach's own load already holds the record.
- `updateRunState`, `Run.Stop`, `stopRecorded` and `updateRelaunchRecord`: the terminal-record case. **Correct.** `errLeaveRunRecord` writes nothing and the strand is still removed.
- `SwitchAccount`: hub pin, beams pin with no entry, beams pin with entry, sibling switch, one account of positive weight, unlisted launch. **Correct.**
- `pickAccount`, `resetCounts`, `lowestLoad` and `earliestUnblocked`: weights 2 and 1, an added account, a reweighted account, every candidate short. **Correct.** The counts and tie-breaks match D3.
- `PickAccount`: nil maps, `keepEntry`, `Names` failing, no candidate. **Correct.** `delete` on a nil map is a no-op.
- `resolveAccount`: hub pin over beams pin over entry; beams pin yielding only with an entry; retired or unlisted entry. **Correct.**
- `refusedResume` and `retryAPIError`: copied resume, first turn end an API error, usage limit, second turn end. **Correct** for the launch offset. The per-`Wait` `turnEnds` reset after an attach is a minor risk (F4).
- `readLoadTurn`, `loadSkillTurn`, `switchedAtLimit`: one ask per read, refused-resume count. **Correct.**
- `dispositionCandidate`, `strandRemovable`, `StopStepRuns`, `sweepOrphans`, `unfinished`, `runIsLive`, `readRecordedRunSessionState`: relaunching within the bound, past it, and not tracked. **Correct.**
- `mergeUsage`, `parseStatusLine`, `RecordStatusLine`, `recordBlocked`: later reset, equal reset with higher percentage, earlier reset, out-of-range percentage, reset too far ahead, `blocked_until` kept. **Correct.**
- `CarryTranscript` and `copyTranscript`: transcript only elsewhere, newer elsewhere, resolved newest, none found. **Correct.** The transcript is written last and the source stays.
- `orchcli.relaunchOrch` and `relaunchStrand`, `orchengine.restartOnTurnEnd`: resume, fresh, failed restart. **Correct.** Fresh clears the session id, no watcher is spawned, and a failed restart ends the watcher naming `lym beam start`.
- `reedengine.EndStrandProcess`, `RelaunchStrand`, `planResumeLaunches`: a held non-live strand with a complete done-when list. **Correct.**

**Findings**

- **F1 — MEDIUM, design.** A known config directory with no `sessions/` directory breaks every reader that goes through `readRegistry`.
  - Location: `internal/shuttleengine/claudeengine/resume.go` (`checkResumable` registry loop, `sessionHolder`), `internal/shuttleengine/claudeengine/prober.go` (`processLiveness`). Evidence: `resume.go:302-306` (`readRegistry` returns the `ReadDir` error unchanged), the diff's per-directory loops in `resume.go`, and `registryDirs` in the same file.
  - D8 reads every known directory's registry as one set. The known directories always include `~/.claude`, retired accounts, and a listed account that has never run.
  - `checkResumable` turns an unreadable registry into a warning. `relaunchSpec` (`accountrelaunch.go`) treats any warning as "a live holder could not be ruled out" and refuses. So the first switch onto a never-used account, or a hub with both accounts off `~/.claude` and no `~/.claude/sessions`, ends the run as died.
  - `sessionHolder` returns an error when no live holder is found. `processLiveness` can never read dead, because the missing directory counts as doubt.
  - Fix: treat a not-exist registry directory as an empty registry in the multi-directory readers (`errors.Is(err, fs.ErrNotExist)`), and keep it unreadable for other errors. Add rows to `TestCheckResumable`, `TestSessionHolder` and `TestProcessLiveness`: a live entry in one directory and a non-existent second directory reads alive, not unproven.
  - The existing rows `secondNoDir`, `secondMissing` and "dead entry beside an unreadable second registry is unproven" pin the opposite behaviour; they need correcting.

- **F2 — LOW, design.** `validateAccounts` accepts `~/x` as `/x` when the home directory is unresolvable.
  - Location: `internal/shuttleengine/claudeengine/accountconfig.go` (`dir = filepath.Join(home, strings.TrimPrefix(dir, "~"))`). Evidence: `config.go`'s LoadConfig comment, "An unresolvable home leaves it empty, so a `~/` config directory is refused as not absolute".
  - `filepath.Join("", "/x")` is `/x`, which is absolute, so the refusal never happens.
  - Fix: return the refusal explicitly when `home == ""` and `config_dir` starts with `~`.

- **F3 — LOW, design.** A failed `ReleaseStrandHold` after a failed `EndStrandProcess` leaves a live strand held.
  - Location: `internal/shuttleengine/stop.go` (`stopForRelaunch`, the release-failure log). Evidence: `reedengine/hold.go` (`EndStrandProcess` saves `Held` first) and `lifecycle.go` (`planResumeLaunches` skips held).
  - The failure is only logged and the run goes back to running. If the strand later dies, reed's Resume neither relaunches nor drops it, and nothing clears the hold. Only `Wait`'s died path or a stop removes the strand.
  - Fix: return the release failure joined onto the returned error, or have `endFailedRelaunch` and the `Wait` died path remove a held strand whatever its mark. State the intended owner of the hold in the reed doc.

- **F4 — LOW, design.** `apiErrorWatch.turnEnds` restarts at 0 on each `Wait` or attach, while `RunState.TranscriptCopied` persists.
  - Location: `internal/shuttleengine/apierrorretry.go` (`retryAPIError`, the `watch.turnEnds++` / `first` block).
  - The attach cursor starts at `PromptOffset`. For a copied resume whose first turn succeeded before a later attach, the next API-error turn end read after an attach counts as "first" and the session restarts fresh.
  - This applies only when turn ends past `LaunchOffset` lie before `PromptOffset`, or when the first turn end was already consumed by the earlier process.
  - Fix: persist whether the first turn end after `LaunchOffset` has been seen (for example advance `LaunchOffset` past the first turn end once read), or count from `LaunchOffset` rather than from the cursor.

**Tool calls**
1. `grep -A22 "^func readRegistry" claudeengine/*.go` — needed to confirm how a missing `sessions/` directory is handled; the diff does not show `readRegistry`.
