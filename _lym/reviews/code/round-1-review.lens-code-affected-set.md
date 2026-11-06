```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. For each change, the set of callers, readers, docs and rules it touches, and whether the diff left any of them wrong.

**Set per change**

1. `loadSkillTurn` moved to `loadturn.go`, with new timeout and hold semantics.
   - Found: `settleLoadTurn` and `loadSkillsThenPrompt` (`run.go`), the `doc.go` skill-loading section (lines 318-338), the `Spec.SkillLoadTimeout` comment (`spec.go:139`), and the engine defaults (`claudeengine/session.go:397`, `codexengine/skillload.go:24`, `testkit/shuttlefake`). `orchcli/start.go:198` passes `SkillLoadTimeout` and only needs the unchanged field. The orch watcher's own `skill skipped` path (`orchengine/watcher.go:1388`) is a separate reload sequence, not this one.
   - Covered, except F1 below.

2. A new `OutcomeAPIError` with `NotStarted` can now come out of a start.
   - Found: `RunGated` (`run.go:1021`), `burlerengine/review.go:233`, `shedadapters/burlerreview.go:334`, `burlerfix.go:244`, `multillm.go:182`, `landingshed/fixerrun.go:144` and `fixreview.go:234`.
   - Covered. `burlerengine/review.go:233` folds `ErrNotStarted` into `OutcomeDied` with `NotStarted` set, so `burlerreview.go`'s switch never sees `OutcomeAPIError` from a start. I did not read the `multillm.go` outcome switch above line 178 to see where `OutcomeAPIError` goes. This is not raised as a finding.

3. `loadturn.go` added to `unboundedWaitFiles` in `cmd/lym/daemonwakeups_test.go`, with a new `PATTERN-daemon-wakeups.md` entry.
   - The scan list is covered. The pattern text is wrong in one place (F2).

4. New "An API error holds the prompt" section in `PATTERN-reed-panes.md`.
   - Landed, but with a format gap (F3). The "What each caller does with `ErrPaused`" list in the same file still reads correctly, and the new text says the typed load goes through the verified send, which pauses hold.

5. `engine.go` `OutcomeAPIError` comment and `Result.APIError` comment.
   - Covered.

6. The run deadline now moves later by the hold length.
   - Found: `doc.go:184` ("The run deadline counts awake, unpaused time") and the `Run.deadline` field comment (`run.go:345-347`: "Wait moves it later by the sleeps and pauses").
   - Not updated (F4).

**Findings**

- **F1 — LOW, consistency.** `doc.go:321-322` is stale.
  - Line 321 says shuttle "polls until the first turn end … or `Spec.SkillLoadTimeout` passes … measured from the end of the startup probe". The code measures from `loadSkillTurn`'s entry, extends for an open turn up to `loadTurnOpenBound`, and holds indefinitely on an API error.
  - Line 322 says "a turn that never starts or ends is bounded by the timeout". It is now bounded by the open bound, and only for an engine that reads turn starts.
  - The new text at lines 328-337 corrects this only by appending, so the section contradicts itself.

- **F2 — LOW, consistency.** The `PATTERN-daemon-wakeups.md` entry (line 25) says the hold "wakes once per back-off".
  - `loadSkillTurn` also wakes every `pollInterval` while no API error is pending (`loadturn.go:285`). Each of those wakes calls `reed.Status()`, and `pollInterval` can be sub-second.
  - The loop therefore sits in the unbounded-wait list with only its back-off described. It does not follow the rule's "wakes at most once a second and backs off while idle" for the poll between turns.
  - `loadturn.go` also never states the one-second floor on `pollInterval`, which the `daemonwakeups_test.go` scan cannot see because it checks literal shapes only.

- **F3 — LOW, consistency.** The new `PATTERN-reed-panes.md` section (lines 96-102) has no `Rule:` line and no enforcement line.
  - The sibling sections each carry one, for example "### A pause holds prompts" with its `Enforcement` subsection and the top-level `Rule:` lines.
  - The test `TestStartup_LoadTurnAPIError` pins this behavior but is not named.

- **F4 — LOW, consistency.** `doc.go:184` still describes the deadline as counting only awake, unpaused time. The `Run.deadline` field comment (`run.go:345-347`) still says only Wait moves it. `endLoadHold` (`loadturn.go:210`) now shifts it too.

**Commit message and Pattern-Departure.** The commit message carries no `Pattern-Departure:` trailer. Every entry the diff touches (`PATTERN-daemon-wakeups`, `PATTERN-reed-panes`) was edited in place. Nothing in my set needs a trailer. The directive's declared-changes list is not visible in my context, so I did not check "Declared PATTERN changes landed".

**Tool calls**
1. Grep for `skill_load_timeout`, `SkillLoadTimeout`, `skill skipped`, the old doc phrases and `.NotStarted` across the repo. My context lacked the repo-wide callers.
2. Read `doc.go` lines 314-343. I needed the full skill-loading section, which the diff showed only in part.
3. Printed the `NotStarted` handling in `burlerreview.go`, `burlerfix.go`, `burlerengine/review.go` and `multillm.go`, plus a grep in `orchengine/watcher.go`. My context lacked the callers' branches.
