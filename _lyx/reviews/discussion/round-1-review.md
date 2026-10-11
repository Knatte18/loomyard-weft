---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:94
    summary: D5's belongs-to-this-halt rule rejects every genuine failure record, because Publish writes the record before it returns Stuck and so before the blocked state begins, and the record carries no write time
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:63
    summary: D2's trust claim that a task cannot set what runs in the prime rests on a config guard that only stops a mistaken write, and ignores that the configured command runs the landed tip's own task-editable code and pushes prod
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:109
    summary: D6 does not account for gitrepo's existing rebaseRetryTriggers, which already classify a remote cannot-lock-ref as divergence (pull --rebase in Push and PushCoalesced, ErrPushRejected in PushRebaseFree), and does not say which push functions get the retry, so the never-rebases-or-forces bound is unbacked
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:97
    summary: D5 retries a lock-refused goto or a not-parked resume with no upper bound while suppressing the blocked notice, so a driver that never releases its lock or never parks leaves the orch unwoken indefinitely
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:82
    summary: D4 never defines what ends a Land failure episode, and its own way forward (lyx batten run) is the same batten restart that must not repeat the notice, so a re-run that fails again either goes silent or contradicts the acceptance criterion
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:5
    summary: Goal's list of what wakes the orch omits the Land and teardown failure notices and the ready-task notice that Scope and D4 add
  - id: F7
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:114
    summary: D6's rationale treats every remote cannot-lock-ref as a race that clears on retry, but the same message also covers a permanent ref name conflict, which the retry repeats to the attempt bound under the caller's lock
  - id: F8
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:40
    summary: D1 step 1 leaves open whether the dirty-prime wait applies when pull and deploy are skipped for a landing into another branch
---

### [BLOCKING] D5's freshness rule rejects every genuine failure record

**Location:** `_lyx/discussion/decision-record.md:94` (D5, first bullet), and the criteria at lines 182 and 183.

**Issue:** D5 counts a record as belonging to this halt only when "it was written at or after the child's blocked state began".
In `internal/landingshed/verifygate.go`, `checkVerify` and `checkPublishVerify` call `recordFailure`, which writes the record, and only then return the Stuck reason through `verdict`.
Shed sets the child's blocked state after the producer returns, so the record is always written *before* the blocked state begins.
`verifytree.PublishFailure` also has no write-time field, so the only timestamp available is the file's mtime, and that mtime falls before the state's since.
As written, every real verify failure counts as stale, D5 never fires, and the criterion at line 182 cannot pass.
Matching the HEAD is not enough by itself either: a record left over from an earlier halt at the same HEAD would pass that check.

**Fix:** Restate the freshness test against something written in the right order.
For example, Publish could write a timestamp, or the run's halt/history entry id, into the record (it already writes the target field D5 adds), and D5 could compare that against the halt's history entry.
Another option is to compare against the start of the Publish row's attempt instead of the blocked state's since.
Update the bound at line 105 to match.

### [MEDIUM] D2's trust claim rests on the wrong guard

**Location:** `_lyx/discussion/decision-record.md:63-68` (D2), and the criterion at line 181.

**Issue:** D2 says `lyx config` refuses a write from a pair, "so a task cannot set what runs in the prime".
The guard it relies on, `hubWriteRefusal` in `internal/configcli/hubwide.go`, says in its own doc that it "guards a mistaken write, not a determined agent": it only reads the strand name the caller passes.
More importantly, D2's bound ignores what the command runs.
`./update-plugins.sh` execs `go run ./tools/deploy`, and that is the landed tip's own code, which any task can edit.
`tools/deploy/main.go` also runs a raw `git push origin <head>:refs/heads/prod`.
So once this lands, every merge into the prime's branch automatically runs task-authored code in the prime and pushes a ref outward, with no orch moment in between.
The real trust anchor is the PR-Gate approval of the merged code, not the config key, and D2 never states this.
Its bound (Land row, prime, landed tip, land lock) bounds *when* the command runs, not *what* it can do.

**Fix:** Have D2 say that the deploy command executes code from the landed tip, that it is trusted because PR-Gate approved that tip, and that it may push (the hub's `prod`).
Have D2 call the pair-write refusal a guard against a mistaken write, as `hubwide.go` does, not as what keeps the command out of a task's reach.

### [MEDIUM] D6 ignores the existing classification that catches "cannot lock ref" first

**Location:** `_lyx/discussion/decision-record.md:109-115` (D6), and the criterion at line 187.

**Issue:** `internal/gitrepo/push.go`'s `rebaseRetryTriggers` includes the substring `rejected`.
A remote refusal `! [remote rejected] <ref> -> <ref> (cannot lock ref ...)` matches it, so today:
- `Push` and `PushCoalesced` run `git pull --rebase` on it.
- `PushRebaseFree` returns `ErrPushRejected`, the divergence sentinel.
  `pushRebaseFreeRetryingOnce` (fetch and push again), `pushRebaseFreeLogged` in `coalesce.go` (warn and leave unpushed) and Publish's `pushRejectedReason` all act on that sentinel.

D6 says the retry "never rebases or forces", but it doesn't say the new check runs ahead of that classification.
Without that, in `Push`/`PushCoalesced` the rebase still runs first, and in `PushRebaseFree` the refusal still surfaces as a divergence.
"`internal/gitrepo`'s push functions" also covers `UpdateRemoteBranchLeased` (the package's force-shaped push) and the two delete forms, and D6 doesn't say whether those retry.
The attack-surface bound is therefore unstated for exactly the edge D6 widens.

**Fix:** Name the primitives that get the retry, and say whether the leased force push and the deletes are included.
State that a refusal of the two retried forms is classified, and retried, before the rebase trigger and before the mapping to `ErrPushRejected`, so it never reaches the rebase or the divergence path while attempts remain.
Then say what it surfaces as once the attempts run out.

### [MEDIUM] D5's lock-refused retry has no bound and suppresses the notice

**Location:** `_lyx/discussion/decision-record.md:92` and `:97` (D5), and the criterion at line 184.

**Issue:** D5 sends no blocked notice for a retried halt, and it retries a goto refused under the run lock, or a resume refused as not parked, "at most once per notice probe" with no cap.
The criterion at line 184 says they are "retried until they land".
A driver that holds the run lock and never releases it, or never parks, therefore keeps batten retrying forever with no notice, which turns a halt into a silent stall.
The bound at line 105 ("once per batten run") counts retry episodes, not attempts within one.
D5 also doesn't say whether InnerRun's existing driver-busy notice (`noticeDriverBusy` in `internal/battenshed/notice.go`) covers this loop.

**Fix:** Give the retry loop a stated bound, a time or attempt cap after which the blocked notice goes out with the refusal, or else state that the driver-busy notice applies to it.
Add the matching criterion.

### [MEDIUM] Land failure episodes are undefined against their own way forward

**Location:** `_lyx/discussion/decision-record.md:81-82` (D4), with D1 line 38 and the criterion at line 172.

**Issue:** D4 sends "one notice per failure episode", kept in a marker "so a batten restart never repeats it".
The notice's way forward is `lyx batten run <slug>`, which is a batten restart, and D1 re-runs the row from the top.
If a re-run hits the same failure (the cause wasn't fixed) or a different one, nothing says whether that's the same episode.
If it's the same, the marker suppresses the notice, and a re-run that fails again goes silent.
If it's a new one, a plain restart repeats the notice, which the criterion at line 172 forbids.
InnerRun's markers end an episode on a state or since change (`keepEpisode`), but the Land row has no child state to key on.

**Fix:** Define the episode key for Land and teardown failures, for example the step plus a counter or stamp that each fresh row entry advances.
Restate the criterion at line 172 so it separates a crash-restart mid-notice from a deliberate re-run that fails again.

### [LOW] Goal's wake list disagrees with Scope and D4

**Location:** `_lyx/discussion/decision-record.md:5` (Goal).

**Issue:** The Goal says the orch is woken "only for parent review, review escalations ..., design calls, PR-Gate and halts whose cause or way forward is unknown".
Scope In (line 13) and D4 also wake it for a Land or teardown failure, whose way forward is known and named (`lyx batten run <slug>`), and for the ready-task notice.
D4 line 87 already calls the ready notice a temporary departure, but the Goal still claims the closed list.

**Fix:** Reword the Goal's list to include land and teardown failures and the ready notice, which is temporary until `orch-split`.

### [LOW] D6's premise that every "cannot lock ref" is a race

**Location:** `_lyx/discussion/decision-record.md:114` (D6 rationale).

**Issue:** A remote "cannot lock ref" also covers permanent conditions, such as a ref name that conflicts with an existing ref (`'refs/heads/a' exists; cannot create 'refs/heads/a/b'`).
D6's matcher ("a message holding cannot lock ref") retries those too, up to `pushMaxAttempts` with doubling backoff, while the caller's lock is held.
The attempt bound keeps this finite, but the rationale states a false premise.

**Fix:** Either narrow the match to the race form, or state in the bound that a permanent ref conflict is retried to the attempt cap before it fails.

### [LOW] Dirty-prime wait when pull and deploy are skipped

**Location:** `_lyx/discussion/decision-record.md:40-41` (D1 step 1), and the criteria at lines 170 and 173.

**Issue:** Step 1 skips pull and deploy for a landing into another branch, and in the same step waits on a dirty prime.
It doesn't say whether a skipped landing still waits up to ten minutes and can go Stuck on a dirty prime, even though it never touches the prime's tree.

**Fix:** State whether the wait applies only when the pull will run.

## Focus verdicts

- D2 and D5 attack surface: D2's trust claim does not hold as stated (F2).
  D5's once-per-run bound holds against batten's resume-not-reseed behaviour (`internal/battencli/cli.go`: no re-seed flag, a fresh status only when none exists).
  Its freshness test is broken (F1), and its inner retry loop is unbounded (F4).
- D1 and D4: the land lock is an flock, released when its holder dies, so the unbounded wait is bounded by the holder's row, which is itself bounded by the dirty-prime cap and the deploy timeout.
  The ready snapshot is sound: `Store.Prune` strips removed slugs from survivors' `depends_on`, so the stored-`depends_on` diff in D1 step 6 is meaningful.
  The failure-notice episode is the open gap (F5), and F8 is a minor ambiguity.
- D6: F3 and F7.
- Relocation: no Decision or Auto-mode assumption is narrative the plan writer doesn't need.
  D1's paragraph at line 50 explains a requirement that the criteria at lines 178 and 179 pin, and it is short enough to keep.
- Goal, Scope, D4 and D8: F6.
  D8's duties don't contradict Scope Out: the orch starting ready tasks is the status quo that `orch-split` replaces.
- Open risks: none is already resolved by a Decision, and no Decision rests on an unresolved risk.
