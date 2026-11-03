```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged each proposal the fixer carried out as written (F1, F2, F4, F5, F6, F8, F9, F10, F11).

**MEDIUM: keying on the attempt token breaks probe and resume of a retry (F2, D1).**
- **Changed text:** "the builder derives it, and the outputs, from the review path and the round's attempt token the review row passes (`1`, then `1b` on its retry), so a probe after an interruption names the same files". Also the D1 acceptance bullet "a retry's paths carry its own attempt token".
- **Summary:** a probe of a round always uses the token `1`. If the interrupted reviewer was attempt `1b`, the probe derives paths for attempt `1`. It then finds attempt 1's old marker, or nothing, never the live `1b` assistant.
- **Evidence:** in `internal/shedadapters/burlerreview.go`, `probeLiveReview` sets `opts.Round = strconv.Itoa(round)` before `ProbeReview` and `ResumeReview`. Only the attempt loop sets `attemptToken += "b"` (lines 254–259). D1 needs either the attempt recorded in the review row's per-round record, which D2 already reads for probe and resume, or a probe of both tokens.

**MEDIUM: the new D6 ending for a dropped first turn contradicts shuttle's startup rule (F8, D6).**
- **Changed text:** "when Codex stays alive but its first turn never starts, as when the positional prompt is dropped, no `SessionStart` arrives, the run ends at its timeout and the marker reads `timeout`."
- **Summary:** the launch declares `RecordsSessionStarts`. With no session-start line, shuttle never marks the run started. The run therefore stays in startup and fails as not started. It does not end at the run timeout as a started run, so the marker would read `not started`, not `timeout`.
- **Evidence:** `internal/shuttleengine/wait.go:1365` says "For a record with RecordsSessionStarts, a ready pane marks the run started only once the events file also holds a session-start line". `internal/shuttleengine/doc.go:546` says such a launch "starts only once the events file holds a session-start line … beside the ready pane". The reviewer's proposed wording was wrong here; the original "not started" reading was closer. What still needs checking is whether shuttle retries such a start.

**LOW: `/tmp` as the report root is tested on Linux only (F1, D2).**
- **Changed text:** "The assistant writes its report under `/tmp`, a writable root of its `workspace-write` sandbox".
- **Summary:** D13 keeps Codex seats on Linux and macOS. On macOS, `/tmp` is a link to `/private/tmp`, and Codex keys a linked path by its resolved target. Nothing tested shows that the sandbox's writable root covers `/tmp` there.
- **Evidence:** spike `report.md:10` (linked paths keyed by the resolved target) and `:283` (writable roots, recorded on Linux only). Twill spike setup line 4: "Linux, tmux 3.6".

Checked with no finding:
- **F10:** the no-parent stencil reads "No parent is recorded for this run. When you are blocked, write your report and stop.", which fits an assistant that sends no message.
- **F4, F5, F6, F9, F11:** I found no case where the proposal itself is wrong.

Tool calls:
1. One Bash call read three things my context lacked:
   - `parent-directive-none.md`, to test whether the no-parent text fits an assistant;
   - a grep of shuttleengine for `RecordsSessionStarts`, to test the new D6 ending;
   - `burlerreview.go` lines 400–440 (`probeLiveReview`), to test attempt-token keying under probe.
