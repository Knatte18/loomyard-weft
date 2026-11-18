```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that the fixer carried out (F20, F22, F23, F24, F25, and F21's keep-the-GUID option), and build cases where they go wrong.

Findings:

1. MEDIUM. Changed text: D7, step 4 ("re-reads the run record … when a stop verb call has made the record terminal meanwhile, it removes the strand and writes nothing"), together with "Any other refusal, or a failure in steps 2 to 4, ends the run as a strand that died".
   Summary: the F20(4) proposal puts the re-read only on step 4's success path, so the failure path can still overwrite a stop.
   Case: a `StopStrand` lands during the window. `removeStrandIfPresent` removes the held strand when it is own-window and present (`strandRemovable`, `internal/shuttleengine/stop.go:135-136`). Step 4's same-GUID relaunch then fails on a GUID reed no longer tracks, and the failure path ends the run as died over the recorded stopped. The overwrite F20 aimed to close survives, and the new criterion "a `StopStrand` during a relaunch leaves the run stopped" fails in this case.

2. MEDIUM. Changed text: D7, "A check refused for no transcript runs steps 2 to 4 again with a fresh launch: a new session id, the run's skills and the run's prompt", with step 4 changing only "the strand's stored `Cmd` and `ResumeCmd`".
   Summary: keeping the GUID on a fresh relaunch leaves the strand and the run record naming the abandoned session.
   Evidence: reed stores a strand's `SessionID` (`internal/reedengine/strand.go:413`) and its name-drift repair reads it (`internal/reedengine/namerepair.go:178`). Neither step 4 nor the fallback says that reed's `SessionID` or `RunState.SessionID` is replaced. The usage record, the drift watch and the live readers would then follow the dead session.

3. MEDIUM. Changed text: D6, "The resume check reports whether it copied. On the start path the runner carries that fact into the launch…"
   Summary: the copy fact is lost when the orch's resume runs the check twice.
   Case: `resumeOrch` calls `StartSession` once, and on `SessionHeldError` ends the holder and calls it again (`internal/orchcli/start.go:808-816`). The first check copies the transcript and is then refused for the live holder, since the copy runs before the holder test. The second check finds the resolved directory already newest and reports no copy. So the launched run records no copy, and the refused-resume fallback never fires. This is the `lym beam start` path over a live orch, which the new D6 criterion "an orch resume through `StartSession` whose check copied … record the copy" covers.

4. MEDIUM. Changed text: D7 step 2, "no skills, since the resumed transcript holds their loads".
   Summary: dropping the skills from the relaunch spec also changes the appended system prompt, not only the first turn.
   Evidence: `Prepare` builds the subagent rule passage from `spec.Skills` (`subagentPassage(cfg.Agents, spec.ConfigRole, spec.Skills)`, `internal/shuttleengine/claudeengine/claudeengine.go:269`) and writes it into the append-system-prompt file. A relaunched session with no skills loses or changes the passage that tells its subagents which skills to load. Separately, `Start` re-resolves skills from `ConfigRole` (`internal/shuttleengine/run.go:576-582`), so the relaunch has to skip that resolution, and the text does not say so.

5. LOW. Changed text: D7 step 1, "A failure to end the process clears the mark back to running over the live strand", with "held from reed's reconcile and resume until step 4".
   Summary: no path releases reed's hold when step 1 fails after the hold is set.
   Case: the process is ended or the hold is set, then the reed op errors. The record goes back to running, but the strand stays held from reconcile and resume with nothing to release it.

6. LOW. Changed text: D7's orch paragraph, "a fresh start does the same with a fresh launch".
   Summary: `launchOrch` resumes whenever `st.SessionID` is set (`internal/orchcli/start.go:773`), so the fresh start must clear the recorded session first, and the text does not say so. Without that, the "fresh" start resumes the refused session again.

Calls made:
- Grepped `internal/reedengine` for `.SessionID` uses, to check whether keeping the GUID across a fresh relaunch leaves a stale session id in reed; my context did not show reed's use of the strand's session id.
