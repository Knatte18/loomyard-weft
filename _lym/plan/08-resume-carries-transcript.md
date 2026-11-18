# Card 8 — A resume first brings the newest transcript into its account

**Create:**
- `internal/shuttleengine#TranscriptCarrier`
- `internal/shuttleengine/claudeengine#Claude.CarryTranscript`
- `internal/shuttleengine/claudeengine#copyTranscript`
- `internal/shuttleengine/claudeengine#TestClaude_CarryTranscript`
- `internal/shuttleengine#TestRunnerStart_Resume_CarriesTheTranscriptFirst`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.CheckResume`
- `internal/shuttleengine/claudeengine#checkResumable` -> `func checkResumable(sessionID, projectDir string, registryDirs []string, alive func(pid int) bool, startTime func(pid int) (string, bool)) (warning string, err error)`
- `internal/shuttleengine/claudeengine#TestCheckResumable`
- `internal/shuttleengine/claudeengine#TestCheckResumable_UnstattableTranscriptRefusesNamingTheStatFailure`
- `internal/shuttleengine#Runner.checkResume` -> `func (r *Runner) checkResume(spec Spec, engine Engine) (string, bool, error)`
- `internal/shuttleengine#Runner.start`
- `internal/shuttleengine#Runner.launch` -> `func (r *Runner) launch(spec Spec, gate GateSpec, engine Engine, engineName, resumeWarning string, carried []string, transcriptCopied bool) (*Run, Result, error)`
- `internal/shuttleengine#Run.relaunchForReprompt`
- `internal/orchcli#orchCLI.resumeOrch`
- `internal/orchcli#TestStart_ResumeOverALiveHolder`
- `internal/shuttleengine/claudeengine#TestClaudeProjectDirFor_EncodesNonAlnumBytes`
- `internal/shuttleengine/claudeengine/doc.go`

**Delete:**
- `internal/shuttleengine/claudeengine#claudeProjectDirFor`
- `internal/shuttleengine/claudeengine#projectsRoot`
- `internal/shuttleengine/claudeengine#sessionRegistryDir`

**Extends:**
- `internal/shuttleengine#RunState`
- `internal/shuttleengine#Spec`
- `internal/shuttleengine#SessionHeldError`
- `internal/shuttleengine#resumeFakeEngine`
- `internal/shuttleengine/claudeengine#writeSessionTranscript`
- `internal/shuttleengine/claudeengine#replaceTranscriptWithDirectory`
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine/claudeengine#Claude.resolveLaunchAccount`
- `internal/shuttleengine/claudeengine#Claude.knownConfigDirs`
- `internal/shuttleengine/claudeengine#newestTranscript`
- `internal/shuttleengine/claudeengine#projectDirUnder`
- `internal/shuttleengine#TestRunnerStart_Resume_ChecksSessionAgainstPaneCwdBeforePrepare`

**Intent:**
D6's copy and its report.
`shuttleengine.TranscriptCarrier` is an optional capability: `CarryTranscript(sessionID, workdir string) (copied bool, err error)` brings the session's newest transcript into the account the resume runs under.
`Runner.checkResume`, for a spec with `ResumeSessionID`, calls it first when the engine implements it, then the `SessionResumer` check as today, and returns whether it copied, ORed with `Spec.TranscriptCopied`.
A carry failure refuses the resume like the check's own refusal.
On a `*SessionHeldError` it sets the error's new `TranscriptCopied` field, since the copy happened before the holder test refused.
`Runner.start` passes the fact to `launch`, which writes it into the new run record's new `RunState.TranscriptCopied` (`transcriptCopied`) when the record is first saved, after the run directory exists.
`RunState` also gains `LaunchOffset` (`launchOffset`), the events file's size at the run's latest launch, zero for its first; card 16 counts a resume's first turn end from it.
`relaunchForReprompt` writes the copy fact into its record's `TranscriptCopied`, and its startup events offset into `LaunchOffset`, with the new strand.
After card 7 and this card, `claudeProjectDirFor`, `projectsRoot` and `sessionRegistryDir` have no caller and are deleted; `TestClaudeProjectDirFor_EncodesNonAlnumBytes` is retargeted to `projectDirUnder`, and the file doc comments of `resume.go` and `transcriptread.go` say they read every known directory.
`Spec.TranscriptCopied` lets a caller that retries a resume after a live-holder refusal carry the first check's report into the retry: `orchCLI.resumeOrch` sets it from the held error before its second `StartSession`.

`Claude.CarryTranscript` resolves the account by D4 (`resolveLaunchAccount`), finds `<session>.jsonl` under the worktree's project in every known directory (card 7), the resolved one included, and when the newest copy is not the resolved directory's copies it over the resolved directory's, with its `<session>/` directory when present, leaving the source in place; `copyTranscript` creates the resolved directory's project directory when the account never ran in the worktree.
Found nowhere, it copies nothing and reports false; `CheckResume` then refuses with its no-transcript error as today, and the caller starts fresh or fails as it does today.
`CheckResume` stats the transcript in the resolved directory's project, and `checkResumable`'s live-holder test reads the registries of every known directory as one set, as `sessionHolder` does after card 7.
`claudeengine/doc.go`'s resume paragraph is rewritten to match: the no-transcript refusal names the resolved directory's project instead of the pane cwd's project directory, and the resume check reads every known directory's registry as one set, as `SessionHolder` and the prober do after card 7.

Tests are tier0:
`TestClaude_CarryTranscript`, on fake directories: a transcript only under another known directory is copied with its session directory into the resolved one, reports true, and the source stays; a resolved directory holding an older copy gets the newer one; a resolved directory holding the newest copies nothing and reports false; a resolved account with no project directory for the worktree gets it created and the copy; a transcript nowhere reports false and `CheckResume` then refuses with `ErrNoTranscript`; with the prime's account changed by a beams pin the copy lands in the pinned directory, so the orch's resume passes and does not launch fresh (D6).
`TestCheckResumable` gains a row: a session registered live in the second of two registry directories refuses.
`TestRunnerStart_Resume_CarriesTheTranscriptFirst`, on `resumeFakeEngine` extended with a carry result and call order, on the setup of `TestRunnerStart_Resume_ChecksSessionAgainstPaneCwdBeforePrepare`: the carry runs before the check and before `Prepare`; a copy is recorded as `transcriptCopied` in the new run record, through `Start` and `StartSession`; a `Spec.TranscriptCopied` retry records it though its own carry copied nothing; a held refusal's error carries `TranscriptCopied`; a carry that fails refuses the resume before any run directory is created.
`TestStart_ResumeOverALiveHolder` asserts the retried spec carries the held error's copy fact.

**ImpactSummary:** The runner's private resume check gains a result its two callers take; three shuttle types gain one tolerant field each; the claude check widens to every known registry; orch's retry passes one flag.

**Commit:** 8: resume: carry the newest transcript into the resolved account and record the copy
