```yaml
lens: interactions
outcome: reported
```

Interactions lens: I checked each pair of decisions that act on the same state. Five findings: 2 MEDIUM, 3 LOW.

**Pairs checked**
- D1 × D2 (an empty list still reads a record; an invalid list fails the launch): held for launches.
- D1 × D6 (an invalid list against the readers' set of known directories): broken, finding 5.
- D1 acceptance × D2 acceptance (resume with no record, list empty or listed): broken, finding 4.
- D2 × D3 (one write lock, the re-read under it, the search before a pick): held.
- D2 × D4 (relaunch replays the stored command; the open launch follows the fresh rules): held.
- D2 × D5 (the record's stored name against the status line's current-name lookup): held. The stored name is never read.
- D2 × D6 (resume with no record against the resume check's fallback, and shuttle's check-before-Prepare order): broken, finding 1.
- D2 × D7 (the search's session-id sources against where the orch keeps its resumable id): broken, finding 3.
- D3 × D5 (eligibility read from per-name usage files, the merge rule, a rename or reused name): broken, finding 2.
- D3 × D7 (prime and beams pick when they have no transcripts): held.
- D3 × D8 (fallback when every account is short; limits are waited out): held.
- D4 × D6 (a record wins in both the pane and the readers; without one, the `env` module comes first): held.
- D4 × D5 (an opened session gets a status line only when settings are written, i.e. with a board dir): held. `command.go` OpenLaunch writes settings only when `creelFlagDirForHook != ""`.
- D5 × D9 (the manual check expects one usage file per used account): held.
- D8 × D2 (a limit wait, then a resume under the recorded directory): held.

**Findings**

1. **MEDIUM · design · D2 resume bullets × D6 resume check.** A resume with no record never reaches D2's search of every known directory. Shuttle runs `CheckResume` before `Prepare`. With no record, D6's check falls back to the `env` module's directory, then the process's, then the default. If the transcript is in a listed account's directory, for example after the operator removed a record, the check refuses with "no transcript" before D2 can find and record that directory. The same order means a resume whose recorded directory is gone gets the check's generic refusal, not D2's refusal naming the directory and the way forward.
   - Evidence: `internal/shuttleengine/run.go` `start`, where `checkResume` runs at offset +28 and `engine.Prepare` at +89; `run.go:969`; `resume.go:29-39`.
   - Fix: D6's resume check must search every known directory when there is no record, and D2's gone-directory refusal must live in the check.

2. **MEDIUM · design · D5 merge rule × D5 rename claim × D3 eligibility.** When a name is reused for another directory, the old account's usage file stays under that name. The merge drops any incoming window with an earlier `resets_at`. So the new account's real figures are dropped until the old reset passes, up to seven days. D3 then passes the new account over on figures from the old account. That contradicts D5's claim that a reused name "files its figures under the entry that names the directory now".
   - Evidence: decision record D5 lines 166-168 and 190-192; D3 step 3.

3. **LOW · design · D2 search sources × D7.** D2's search reads session ids only from shuttle run records and reed strands (`reedengine/state.go:29`). The orch resumes from its own state record (`orchengine/state.go:88`, read at `orchcli/start.go:713`). If the orch strand has left reed's state, a fresh launch in the prime can pick another account. The orch's resume then fails the check and `launchOrch` silently starts a fresh session in its place (`orchcli/start.go:658`). That breaks D7's promise that the first launch records the directory holding the pre-change transcripts.

4. **LOW · consistency · Acceptance criteria D2 vs D1.** The D2 criteria "a resume launch in a worktree with no record … records the default directory" and "… under the `env` module's `CLAUDE_CONFIG_DIR`, records that directory" do not say the list is non-empty. With an empty list, the D1 criterion and D2's first bullet say a resume writes no record, so read as written the criteria contradict each other.
   - Evidence: decision record lines 66, 272, 277 and 281.

5. **LOW · consistency · D1 invalid list × D6 readers.** D1 says an invalid list fails the launch and never degrades to no accounts. D6's known directories include "the listed accounts' directories", but the record does not say what the liveness read, `lym report` or the resume check do when the list fails to load.
   - Evidence: decision record lines 50 and 203.

**Tool calls**
1. `run.go` `checkResume` body and grep: the order of the resume check and `Prepare` was not in my context.
2. Grep of the orch and dobby state for a session id: where the orch keeps its resumable id was not in my context.
3. `run.go` `start` offsets and `orchcli`/`orchengine` state: to confirm the check-before-Prepare order and the orch's resume source.
4. Grep of `reedengine` state for a session id: to tell whether reed strands hold session ids, which D2's search relies on.
