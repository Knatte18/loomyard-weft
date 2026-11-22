---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed-differently
    reason: "The review proposed widening the blast radius to name the producers and the bouncer, but the field takes one word, and `shedadapters` drops burlerengine, whose audit the diff also changes. The word stays `burler`, which spans both packages, and the section's prose names the record's readers. The door is now one-way as proposed. The edit is in `_lym/landing/summary.md`, which is not committed, since no git command may touch `_lym`."
---

# Fixer report — burler code review, round 1

## Changes

Four commits, all green under `lym gate test ./internal/burlerengine ./internal/shedadapters`.

`d45d24d3` — `internal/burlerengine/cluster.go` and `cluster_test.go` (F1, F2, F3, F4, F5, F8, F9, F11):

- F1: `setRemote` never turns a remote that stopped leading in back to leading in.
  `set-url --push` and `--add` keep a remote leading in only when it did and the URL does.
  `set-url --delete` never keeps it.
  A remote change under an unknown directory marks every push redirected.
  A worktree's remotes are keyed under the repository it was added from (`worktreeSources`, `configRoot`).
- F2: `parseGitInvocation` reads the attached `--config-env=` form, and `include.` and `includeif.` are redirect key prefixes.
- F3: `leadsInto` resolves a relative path from the top level of the created repository the call runs in, and a relative path under no created repository leads nowhere.
  A clone's source keeps resolving from the clone's directory through the new `sourceLeadsIn`, since git runs a clone in no repository.
- F4: `forkViolation` takes `pushFrom` from every fork's commands, and the push way forward says the push went to a destination the audit could not place.
- F5: a clone scans only its `-c` and `--config` values for a redirect key (`cloneConfigValues`).
- F8: any `ln` marks every push redirected.
  `record` refuses a directory that an earlier git call entered outside every created repository, or an ancestor of one (`enter`, `enclosesEntered`).
  Known gaps in `_lym/landing/summary.md` name the symlink limit.
- F9: a push from outside every created repository with `-u`, `--set-upstream` or `--recurse-submodules` counts.
- F11: a row for each branch the review named, and a row for each of the above.

`42ac74ab` — `internal/shedadapters/burlerfix.go` and `burlerfix_test.go` (F6, F7, F10, F12):

- F6: `archiveReport` records the halt even when it archived no report, and `recordRerun` keeps the report the record already holds for the same digest.
  An attempt 2 that archives nothing replaces the reason and keeps the report.
- F7: rows for a pre-attempt record that survives an exit recording nothing, and for the leftover report a resumed session left (`TestBurlerFixProducer_Call_TellsAFixerOverAResumedSessionsReportWhy`).
  The "that session never finished the round" case is reachable through a resumed fixer that dies, so the switch case stays.
- F10: the cancellation row asserts the recorded reason and report.
- F12: `archiveWholeRound` clears the rerun record, and the gate-failed test asserts it.

`cc675bc3` — the self-check's accepted findings (see below).

A fourth commit repairs the gate's comment findings: it breaks the wrapped comment lines of `cluster.go`, `burlerfix_test.go` and `roundsnapshot_test.go` at sentence boundaries and splits the over-long ones.
No code changed in it.

`_lym/landing/summary.md` also carries the changes for F13 and the changed behaviour above; it is not committed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW: `roundsnapshot.go` doc comments stale about what the record keeps — fixed.
- fix-consistency, LOW: blast radius `shedadapters` names less than the diff can break — fixed, by keeping `burler` (see F13).
- fix-consistency, LOW: `Call` doc silent about the failure exit's recording — fixed.
- fix-proposal, LOW: any `ln` makes every later push count — disputed, every fix worse.
- fix-affected-set, LOW: the review producer's `archiveRound` retires a round without clearing the rerun — fixed (`clearFixerRerun`, a test in `roundsnapshot_test.go`).
- fix-affected-set, LOW: an attempt 2 that archives nothing records nothing before an exit that records nothing — disputed, every fix worse.

## Disputed

- Fork finding fix-proposal LOW (`ln`), ground `every fix worse`.
  A narrower rule that flags only an `ln` whose operand lies inside a created root misses `ln -s /srv/target /tmp/x`, then `git init --bare /tmp/x/b.git`, then a push to `/tmp/x/b.git`: the link is made before the root exists, and the push then reaches the target.
  Evidence: `internal/burlerengine/cluster.go:201` (the `ln` case) and `internal/burlerengine/cluster.go:283` (the redirected check).
- Fork finding fix-affected-set LOW (attempt 2 records nothing), ground `every fix worse`.
  The only exit that records nothing is a fixer that could not be stopped, which returns bare so that nothing is written about a session that may still be running.
  A record there would also name a report that session may still be writing.
  Evidence: `internal/shedadapters/burlerfix.go:372` (the `ErrHalfNotStopped` case of `errorExit`).

## Deferred

Nothing deferred.

## Rulings

No parent ruling reached this round.
