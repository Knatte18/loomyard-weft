```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness over the diff. For each changed function or branch, the case I ran and whether it is correct:

- `retryAPIError` (`internal/shuttleengine/apierrorretry.go`), `model_not_found` with `APIErrorRetries` 0 or 2 and no output files: `watch.rejected` is true, `outcome` is `api-error`, and `endOnAPIError` returns the result at once. **Correct for the outcome.** Strand cleanup is a separate defect, F1 below.
- `retryAPIError`, rejected model but every output file exists: `classifyDeadlineExpiry` gives `done`, `endOnAPIError(done, …)` finalizes `done`, and `APIError` stays empty. Correct.
- `retryAPIError`, rejected error followed by a turn start, turn end or session start: `rejected` resets to false together with `pending`, so a stale rejection cannot end the run. Correct.
- `retryAPIError`, an API error that is not a rejection, with `APIErrorRetries` 0: the events file is now read, `watch.errors` counts, and the function returns `false`. Same behavior as before, except the events are read for every run now. Correct.
- `retryAPIError`, a rejected error while a pause covers the run: the `covered` gate is outside the diff and I did not re-read it. The doc comment says a covered run is held. Not verified; no finding.
- `signalFromPayload` `StopFailure` (`claudeengine/signals.go`): `error` is `"model_not_found"` gives `ModelRejected` true; a missing or non-string `error` gives `""` and false; `"rate_limit"` gives false. Correct.
- `validateSpec` / `validateChoice` (`codexengine.go`): same checks and same order as before, version first and then permission mode, then effort. The resumed-session refusal now comes before them, so only the order of refusals changed. Correct.
- `OpenLaunch`: a valid spec returns `Cmd` with an empty `ResumeCmd` and `SessionID`. `Runner.Open` chains `ResumeCmd` through `chainStatements`, which drops the empty part, so the resume line becomes `touch marker; reed remove; exit` and the strand ends. Correct.
- `OpenLaunch`, a relative or empty anchor, `.codex/rules` present, or a `prepareHome` failure: `readyHome` returns the error before anything is written, and `Runner.Open` creates nothing before `OpenLaunch`. Correct.
- `OpenLaunch` and `loadConfigOrRefuse`: it does not call `Prepare`'s gate-config load, which an open has no use for. Correct.
- `readyHome`: the `Records` symlink-resolve failure falls back to the told path with a Warn. It runs after `prepareHome`, so a refusal still writes nothing. Correct.
- `codexCommand` / `buildLaunchCmd`: with hooks and a first turn the output equals the old line, and the exit is chained after. With no hooks the bypass flag is dropped, and with no first turn the positional prompt is dropped. A `Prepare` line always has hooks, because its event and activity hooks are always present, so the bypass flag stays. Correct.
- `resolveModelSpec` and `run`'s `--model`: an empty `--model` gives a zero `ModelChoice`, so `shuttle.yaml`'s engine runs. A malformed spec gives `ok:false` before any engine prepares. `result.APIError` is added to the envelope only when non-empty. Correct.
- `checkCodexFolders` (`home.go`): only the messages changed, and the presence check is unchanged. Correct.

Findings:

- **F1 — MEDIUM, design.** `internal/shuttleengine/wait.go:1884-1962` (`finalize`), reached from `internal/shuttleengine/apierrorretry.go:74-76` and `:97-107` (`endOnAPIError`).
  - A rejected-model run still leaves its strand alive.
  - `finalize` removes the strand only when `outcome == OutcomeDone && !KeepPane`. An `api-error` ending therefore keeps the Claude pane and the strand registered. The timeout path at `wait.go:439` behaves the same.
  - The decision record's item 1 names the trial fault as "the run sat to its timeout and left its strand alive". The diff fixes the wait and leaves the strand. The `run` CLI (`internal/shuttlecli/run.go`) never stops it either, and my grep found no `Stop` or `RemoveStrand` call there.
  - The focus asks for "no live strand" on both the claude and codex paths. The fix is for the rejected-model branch to end the strand, for example through `removeDiedOwnWindowStrand` or the shuttle stop verb.
  - I read the code only. No test asserts strand removal for the rejected branch.
- **F2 — LOW, consistency.** `internal/shuttleengine/doc.go:75`.
  - It still says a run ends api-error only for "a spec with APIErrorRetries only". `retryAPIError` now ends a run `api-error` with 0 retries on a rejected model.
  - Line 123 of the same file carries the new rule, so the two lines disagree. `wait.go:8` repeats the old wording.
  - The fix is to drop the "only" qualifier at both sites.

Tool calls:
1. `grep` of `internal/shuttlecli/run.go` for `Stop`, `RemoveStrand` and `Outcome`, plus `wait.go:425-445` and the `retryAPIError` call site. My context lacked what the CLI does after `Run` returns and the timeout path's cleanup.
2. `grep` for `Chain` in `internal/shell`, to check how an empty `ResumeCmd` chains. It was not in my context.
3. `grep` for `chainStatements`, to confirm it drops empty parts. It was not in my context.
