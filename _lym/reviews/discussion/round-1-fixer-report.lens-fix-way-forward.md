```yaml
lens: fix-way-forward
outcome: reported
```

Task: check the fix's new refusals, errors, next steps, rows and tests (fix-way-forward lens) to confirm each can be reached from the state it names and leads to progress.

**MEDIUM**: D6's fresh-start fallback names no trigger on the orch side.
- Changed text: "A resume whose first turn ends on an API error the engine classifies as a refused resume starts the session fresh … the orch through its start's fresh launch."
- Problem: nothing named watches the orch's first turn after a resume and then calls a fresh start.
- D7 tells the orch watcher only "a restart capability backed by `orchcli`'s resume path, `resumeOrch`". That path resumes; it never launches fresh.
- The `Watcher` struct (`internal/orchengine/watcher.go:125-132`) holds a session reader, creel, cfg, paths, stencils, skills and a clock, and no starter.
- The only fresh fallback in `orchcli` (`start.go:781`, "this machine holds no transcript … launching fresh") runs on a missing transcript at start. It does not run on an API error after a resume has started.
- Result: the orch half of the fallback cannot be carried out from the state the record names. The D6 acceptance row covers shuttle only.

**LOW**: D6's fallback hangs on a classification nobody defines.
- Changed text: "an API error the engine classifies as a refused resume" and "`retryAPIError` neither counts nor re-sends that turn end".
- Problem: the fix dropped the only stated signal for this case, the 400 "invalid signature" claim, and named no replacement.
- `SessionSignal` carries `ErrorText` (`apierrorretry.go:65`), but no text or rule is given to match on.
- So neither the fallback nor the D6 acceptance row ("an API error classified as a refused resume") has a defined input.

**LOW**: D7's no-transcript fallback skips a step.
- Changed text: "A check refused for no transcript starts the session fresh in the same run."
- Problem: by then step 2 has already prepared a resume launch, so the run directory's records and reed's stored command name a resume.
- Starting fresh needs a second `Prepare` with a fresh launch (a new session id and the run's prompt), and that step is not named.
- The order D7 lays out ends at step 4 with the resume launch.

**LOW**: `set` under a hub pin has no refusal or way forward.
- Changed text: "cannot override a hub pin or a beams pin that is not out."
- Problem: the record does not say whether `set` refuses or silently writes an entry that D4 never reads.
- Under a hub pin that entry can never take effect, and the operator is not told so.
- The `set` acceptance row tests only unlisted or retired accounts.

No further tool calls were made.
