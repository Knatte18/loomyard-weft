# Fixer report — discussion round 1

All six findings fixed in `_lyx/discussion/decision-record.md`; `support-log.md` unchanged.

## Changes

- **F1** (D6, Acceptance criteria): D6 now states that a new post, reply, forward, return or inject keeps a pending confirm's deadline, and defines a confirm as pending until the next successful listing.
  Outside a confirm, new mail restarts the wait at the shorter of the base and the time left, so mail never lengthens the wait; without this, posts before the first miss could also push the release past 60 s (`sweepLoop` in `internal/creelengine/sweep.go` restarts the timer at `sweepBackoffBase` on a changed `mailSequence` without sweeping).
  The Rationale's 60 s is narrowed to successful listings, with each failed listing delaying by at most its own wait; a failed listing while a miss is pending sets the confirm interval.
  The removed-holder criterion names the successful-listing condition, and two rows are added: a post inside the confirm interval, and a post that arrives with less than the base wait left.
- **F2** (D6 first bullet): reworded so the confirm interval replaces whatever wait the first-miss sweep would set, the changed-set base reset included.
- **F3** (D4, Bound, Acceptance criteria): the exit signal carries whether the daemon lost the start race, read from the exit status (`lyx creel daemon` exits 0 on `ErrDaemonRunning` or a clean stop and 1 on a failed start, per `internal/creelcli/cli.go`).
  On that exit the client keeps today's backoff; on any other it makes the one last connect.
  The Bound is restated, and the acceptance line covers a winner that holds the lock but does not yet answer.
- **F4** (D9 Bound): the Bound now names both claude gates the classification covers, lets the bypass gate through because claude shows it only to a strand whose own launch flags request bypass permissions, names the llm smoke test's helper as the seam's only caller, and makes a production caller a new decision.
  It also states that the trust-list side effect follows every call that accepts the folder-trust gate.
- **F5** (D1): the cut test is now "does not parse as a record", and the shared-definition sentence covers the parse test as well as the last line.
  The review's example `{"id":"x"}` does unmarshal into `record`, since `json.Unmarshal` accepts partial objects, so D1 cites `[1]` instead; the premise holds for any valid JSON that is no object.
- **F6** (Scope): added `internal/creelcli/doc.go` and `internal/shuttleengine/doc.go` to the same-commit docs bullet, each with the decisions that change it.

## Disputed

None.

## Deferred

None.
