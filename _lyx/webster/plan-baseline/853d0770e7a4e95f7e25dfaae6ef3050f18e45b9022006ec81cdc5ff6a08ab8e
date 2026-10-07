# Card 1 — payload-authority

**Edit:**
- `internal/shuttleengine/claudeengine/background.go`
- `internal/shuttleengine/claudeengine/background_test.go`
- `internal/shuttleengine/engine.go`
- `internal/shuttleengine/claudeengine/doc.go`

**Intent:**
Fix the #411/#415 root cause (D1, D2): the Stop payload's `background_tasks` becomes the authoritative outstanding signal.

In `outstandingBackgroundTasks`, when the Stop fields carry the `background_tasks` key with a list value, even an empty one, its entries with status `running` are the whole outstanding list and the transcript at `transcript_path` is not read.
The transcript scan runs only when the key is absent or its value is not a list.
Drop the "both signals are always read" rule and the cross-signal dedup that existed for it; keep the per-signal id dedup.

In the transcript fallback, `transcriptHasUnmatchedLaunch` also counts as a completion notification, besides a `type: "user"` message carrying `<task-notification>`:
a `{"type":"queue-operation"}` line whose `content` names the launch's task id, and an attachment of type `queued_command` whose `prompt` names it.
The "after the tool result" ordering rule applies to these lines exactly as to a user message.
Copy the real line shapes from the two transcripts named in the decision record's Notes when writing the test fixtures, finding the lines by tool-use id and task id, since the record's line numbers drift:
in the test-suite-prune-rest transcript the launch is `toolu_0156x6kuzhRxYK53hcQhn2Lk` (line 281, its result at line 283), whose task `bn68egcp9` is absorbed by `queue-operation` lines (293, 297) and a `queued_command` attachment (302);
in the bugfix-sessions transcript the launch result is at line 95 (task `b5o9v6f2u`), absorbed by `queue-operation` lines 356 and 360 and a `queued_command` attachment at line 362.
Trim them to the fields the parser reads.

`shuttleengine.BackgroundTask` gains a `Signal` field naming which signal reported the task, with two exported provider-neutral constants for its values, `payload` and `transcript`, so the trace line D3 adds can name the source.
`payloadTask` sets the payload value and the transcript scan sets the transcript value.
A `subagent` payload entry keeps no command label, but takes its `description` as `Label` when present, so a fork can be named in a notice; the `BackgroundTask.Label` doc in `engine.go` stops saying a fork's label is empty.

Bound (D2), stated in the doc comment: a payload that omits a running task, lists it under another status, or changes an entry's shape under the same key counts as no outstanding work, and the transcript does not correct it.

Tests in `background_test.go`, as rows in the existing table-shaped tests where one fits:
a payload with `background_tasks: []` over a transcript whose launch is matched only by `queue-operation`/`queued_command` lines yields `EventStop` (the #411 shape);
a payload with `background_tasks: []` over a transcript with an unmatched launch yields `EventStop` (payload wins, transcript unread);
a payload with a non-`running` entry yields `EventStop` and no transcript scan;
a payload without the key, and one whose value is not a list, fall back to the transcript, and a `queue-operation` notification matches its launch;
a running shell or fork entry still yields `EventWaiting`, each task carrying `Signal`, and the fork entry carrying its `description` as `Label`.
The existing `TestParseEvents_RunningShellAndUnmatchedForkLaunchBothListed` and `TestParseEvents_PayloadEntryAndLaunchSharingIDYieldOneTask` pin the retired both-signals rule: rewrite them to the new rule rather than adding beside them.

Update the `background.go` and `events.go`-describing paragraphs of `claudeengine/doc.go` to the new rule.

**ImpactSummary:** Changes which Stop lines ParseEvents reports as waiting; adds BackgroundTask.Signal, read by no caller yet.
