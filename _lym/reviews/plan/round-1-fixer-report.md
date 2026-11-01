# Round 1 fixer-report — plan

Every finding's premise held against the cards and the code.
`lym loom validate-plan` passes after the fix and after the self-check edits, with only the two informational `create-new-unit` findings.

## Changes

- **F1 (BLOCKING), fixed as proposed, with one addition.**
  Card 10's Run directory paragraph says `writeLease` and `expireLease` rewrite `lease.json` through `state.UpdateJSON(<lease path>, <lease path>.lock, mutate)`, citing PATTERN-tolerant-records: `writeLease` keeps the later deadline, and `expireLease` writes now less a second.
  `internal/state#UpdateJSON` is listed under **Uses:**.
  Card 11's Front paragraph names the current run id in the record `current.json`, written through `state.WriteJSON`, and `internal/state#WriteJSON` is listed under **Uses:**.
  The addition: `current.json` takes its own lock path, `current.json.lock`.
  `WriteJSON` acquires its lock path itself (`internal/state/state.go:30-43`), and the front already holds the key directory's `lock` at that point.
- **F2 (MEDIUM), fixed as proposed.**
  Card 11 lists `internal/gatecli#TestGateTest_RefusesBeforeResolvingTheHub` under **Edit:**.
  Its Tests paragraph removes the subtest "help names the shared keep-cache-warm cap as the slot wait", the then unused `internal/shuttleengine` import, and the header's clause on the slot wait.
  A file target `//internal/gatecli/cli_test.go` was tried first and refused by `validate-plan`'s `edit-test-file` check.
  The function is the file's only member, so the function target covers the header and the import.
- **F3 (MEDIUM), fixed as proposed.**
  Card 10 drops `TestDetachStore` and `TestStartReading` from **Create:** and from its Tests list.
  Card 11's `TestJoinable` reused-pid row now drives `joinable` with a reader built from `startReading("windows", ...)` over injected creation times.
  That one row pins the Unix-nanosecond reading, no join when `CreationTime` fails, and no join for a different creation time.
- **F4 (LOW), fixed as proposed.**
  Card 14 adds `internal/loomshed#verifyLineFindings` under **Edit:** and `//internal/loomshed/doc.go` under **Uses:**.
  Its Intent rewrites the spec's gate-keys sentence: a Go site runs `--wait`, and an agent's card, batch and `webster-fix` calls carry none.
  It also rewrites the `verify-gate-no-wait` reason in the spec and in the finding's Detail.
- **F5 (LOW), fixed differently.**
  Card 11's Choice (D2) paragraph gains a Bound sentence for the uncapped in-process slot wait.
  The proposal said an interrupt "exits 1 as today".
  The sentence says it exits 128 plus the signal number instead: `acquireSlot` returns 1 on a cancelled context (`internal/gatecli/run.go:558-559`), but `runTest` passes the code through `exitAfterSignal` (`run.go:176`, `617-621`), which answers 128 + n for a caught signal.
  The self-check widened the sentence's list of holders (see below).
- **F6 (LOW), fixed as proposed.**
  Card 13's runner paragraph covers a matched card with no `**Verify:**` line in the runner's read.
  The runner writes the in-process ok envelope to `stdout` and an unmarked exit `{Code: 0}`, and the front replays it.
- **F7 (LOW), fixed as proposed.**
  Under `none`, card 11's `helperLaunch` writes `gate helper: the runner was never started` to the run's `runner.log`, and the `none` row asserts that line on the front's stderr.
- **F8 (LOW), fixed as proposed.**
  Card 4 replaces `internal/shuttlecli#shuttleCLI.awaitCmd` with `//internal/shuttlecli/await.go` under **Edit:**.
  Its Intent rewrites the file header and the `awaitVerb` comment.
- **F9 (NIT), fixed as proposed.**
  Card 8 lists `//internal/runproc/doc.go` under **Edit:** instead of **Extends:**.
  Card 9 moves `//internal/gateslot/doc.go` to **Edit:** and drops the empty Extends group.

## Self-check

Every fork returned a report; the four lens reports sit beside this file.

- fix-consistency, LOW: card 11's new Bound sentence leaves out a slot-held card-verify line as a holder. Merged with the fix-claims MEDIUM below. Fixed as proposed: the sentence names that line's bound, `cardverify`'s per-line timeout.
- fix-claims, MEDIUM: the same sentence names only gate tiers and the build step, but other slot takers share the pool. Fixed with my own change. The sentence now names the bound of each kind of holder: a tier or the build step by `timeout_min`, a slot-held card-verify line by `cardverify`'s per-line timeout, and any other taker, such as a drift step run or a plan-glyph type load, only by its own run, which the card leaves unchanged. I did not take the lens's suggested "bounded by its own timeout", because that would claim a timeout for takers such as `planglyph`'s type load, which acquires with `context.Background()` (`internal/planglyph/typesload.go:126`).
- fix-affected-set, LOW: `internal/shuttlecli/cli.go:9` and `:43` repeat the claim that await needs no config. Fixed as proposed: card 4 adds `//internal/shuttlecli/cli.go` under **Edit:**, and its Intent rewrites both comments.
- fix-proposal, MEDIUM: dropping `TestDetachStore` left no row where a joiner moves the lease later. Fixed as proposed: card 11's `TestGateTest_DetachedRun` gains a row with a 1s cap, a 1s grace and a 4s stand-in. A second call joins before the first lease passes, and the runner is still live after the first call's deadline plus grace.
- fix-proposal, LOW: "the card-verify runner's timeout" in card 14 could be read as the detached runner, which a lease bounds, not a timeout. Fixed as proposed: card 14 now says "`cardverify`'s per-line timeout".

## Disputed

None.

## Deferred

None.
