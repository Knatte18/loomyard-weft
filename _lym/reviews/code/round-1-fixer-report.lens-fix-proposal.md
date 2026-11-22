```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed, and name the cases where a proposal is wrong or breaks something.

**1. MEDIUM — F1: `requiredWayForward` advises `pull_request=merge`, which is not what `required` meant.**
- Changed text: `requiredWayForward` in `internal/dobbyengine/reinterpret.go`, which says to run `lym config beam --layer default --set pull_request=merge` from the prime.
- Case: a hub that used `required` wanted a pull request for every landing, with review. Under `merge`, `ResolvePullRequest` marks `MergeAtOnce`. The pull request is squash-merged on GitHub at once with no review (see the `PullRequestDecision.MergeAtOnce` doc). Following the flagged advice therefore removes the review step.
- It also makes every landing open a pull request, and a task with a missing origin fact halts instead of landing locally.
- Evidence: `internal/landingshed/pullrequestmode.go` shows `mergeAtOnce := mode == PullRequestMerge`. `beamsetting.PullRequestMerge` is documented as "squash-merged on GitHub at once, with no review".
- A fix that keeps `required`'s intent is `pull_request=on` from the prime plus GitHub branch rules.

**2. MEDIUM — F9: dropping a legacy `pull_request: off` flips a task from direct landing to a pull request, with no notice.**
- Changed text: `legacyBeamSettings` in `internal/boardengine/store.go`, plus the `TestLoad` row "a legacy pull_request of off is dropped".
- Case: a hub on the template default `on` that filed a task with `off` to land directly. The old resolver landed it with no pull request. After the drop, the lookup gives `on`, so it opens a pull request and waits for a human at PR-Review.
- The review's own wording ("matches only through the lookup") is incorrect for this case. Nothing mails the orch or logs the dropped value.
- The same applies to the reconcile carry. `LegacyBeamSettings` filters `off` out, so an in-flight run changes behaviour silently.

**3. MEDIUM — F8/F15: an upgrade-era memory is re-read past Land from a worktree that may be gone.**
- Changed text: `unreadLandNoticeRuns` and `rememberRuns` in `internal/dobbyengine/landnotice.go`.
- Evidence: `git show main:internal/dobbyengine/landnotice.go` already has `requested`, `member`, `line` and `clear_owed` as JSON keys. A pre-upgrade record can therefore hold `requested:true` and no `read`.
- Case: such a record is listed past Land. It is "remembered with `Read` false, not `ClearOwed`, not owed", so it lands in `unread`. The lookup then runs against a torn-down task worktree, and the chain falls through to the parent's value.
- `rememberRuns` overwrites `note.Requested` with that reading. If the parent reads `off`, a notice the old binary had decided to send is dropped.
- A migration that sets `Read` when `requested` is true, or that treats a held `Requested` as read, avoids this.
- The proposed `Read bool` is wrong for records that already carry a reading.

**4. LOW — F23: an envsource marker is accepted in worktree files but refused in `run_settings`.**
- Changed text: `expandEnvMarkers` in `tasksetting.go`, against `CheckLocalValues` in `config.go`.
- Case: `pull_request: ${env:X}` in a worktree file now expands in the lookup. The same value in `run_settings` is refused at upsert, because `checkTaskSetting` sees the raw marker. The two paths now disagree on whether a marker is valid.

**5. LOW — F6: sibling scan is an approximation of worktree anchors.**
- Changed text: `settingPlaces` in `internal/dobbyengine/reinterpret.go`.
- It applies the prime's anchor offset to every sibling directory that has a `.git` entry.
- Case: a worktree whose recorded anchor differs from the prime's has its beam files missed. No flag is raised and no carry is written.
- The Bolt checkout is also scanned as a "worktree".
- `TaskSettingChain` uses `NamedWorktree`, which reads the recorded anchor, so the two disagree.

No finding for F4, F10, F14, F20, F22 or F25: I could not build a failing case.

Tool calls made:
1. `git show main:internal/dobbyengine/landnotice.go | grep json:` — to learn which `LandNoticeState` JSON keys the pre-upgrade binary wrote. My context did not hold the old struct.
