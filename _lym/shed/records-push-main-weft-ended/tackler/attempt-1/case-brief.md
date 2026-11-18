# Case brief: records-push-main-weft-ended, stop at Open-IDE (producer)

## 1. The question the next judge must answer

The Open-IDE row's records push reported "the entry of branch main-weft is ended on origin, so the branch was not pushed and is left to its teardown". The commit it was pushing, `b113997e08` ("batten: Open-IDE -> running"), is already an ancestor of origin's `main-weft` tip. Is this push correctly a no-op that should be treated as success, or is there a real unlanded records commit or an ended run entry that needs handling? If the push is meant to proceed, why did `recordsSideNotLanded` fire?

## 2. The facts

- Stop envelope: `_lym/shed/records-push-main-weft-ended/drive-reports/drive-envelope-20261118-113546-7a73.json`. `kind: producer`, `transient: ""`, `first_producer: Worktree-Create`, `current_producer: Open-IDE`, `history_length: 4`, `state: failed`, `hand_back: false`.
- Shed driver report `_lym/shed/records-push-main-weft-ended/drive-reports/drive-report-20261118-113546-e41f.md`: "no rule settles the stop: the tackler decides". No attempts or parent messages recorded.
- Step trace `.lym/shed/records-push-main-weft-ended/steps/f6534f1971f5f535.trace.log` lines 3 to 11: Seed-IDE command (`lym ide seed records-push-main-weft-ended`) exited 0 at 11:35:31.281. Board sync spawned (pid 2357998). At 11:35:31.308 `fabric: mutation kind=commit_created target=/home/knatte/Code/loomyard-LYMHUB/loomyard-weft detail=b113997e08c0cd162c1897b30dffa450cac04697`.
- Step trace line 12 (11:35:45.833): `shed: step refused kind=producer error="fabricengine: push records side at .../loomyard-weft: the entry of branch main-weft is ended on origin..."`.
- Error text origin: `internal/fabricengine/pushanchored.go:309` in `recordsSideNotLanded`. It is the fallback return when the push returns nil and no deferred failed-push record matching the start time exists (`pushanchored.go:297-309`).
- `internal/fabricengine/pushclassified.go:629-` (`goneEntryEnded`) is the separate ended-entry check; `branchGoneOnOrigin` at `pushclassified.go:620` reads `main-weft` as present on origin, so the "gone" path does not appear to apply.
- Read-only git state of `/home/knatte/Code/loomyard-LYMHUB/loomyard-weft`:
  - Current branch `main-weft`, HEAD `181a2955dda5eecc752bfa132609e118fa41dd7b`.
  - `origin/main-weft` = `181a2955dda5eecc752bfa132609e118fa41dd7b`; `ls-remote origin refs/heads/main-weft` returns the same tip.
  - `rev-list --count origin/main-weft..HEAD` = 0 (nothing unpushed).
  - `merge-base --is-ancestor b113997e08 181a2955` exits 0: the commit the stop names is present on origin.
  - `git cat-file -t b113997e08` = `commit` (local object exists).
  - The run's own branch `records-push-main-weft-ended-weft` exists on origin at `c9276efd644ec5100e619d3422ce926774a39697`.
- The stop is a `producer` error, not `blocked`, so there is no stuck reason to name a `goto`.

## 3. What I ruled out, and how

- Commit absent from remote: ruled out. Per the rule, the commit is present because it is an ancestor of the locally known tip (`merge-base --is-ancestor`, exit 0), and `ls-remote` shows the tip equals local HEAD.
- Branch absent from remote: ruled out. `ls-remote origin refs/heads/main-weft` returns a line.
- Unpushed local commits remaining on `main-weft`: ruled out (`rev-list` count 0).
- Transient transport failure: ruled out. The trace shows no transport error; `transient` is empty.
- Plain re-launch as the fix (`resume`): not settled. Local HEAD equals origin and the commit is already landed, so re-running Open-IDE would most likely reach the same `recordsSideNotLanded` fallback and stop again. I did not verify this by running anything.
- `hand-back`: the stop is not a scope or design choice for a parent; it is a code-path judgment.

## 4. Hypothesis (unverified)

Hypothesis: the records push returned nil without actually pushing anything because the commit was already on origin (or the push's deferred-record lookup missed), and `recordsSideNotLanded` (`pushanchored.go:297-309`) then reports a false "ended on origin" error. The message's wording assumes origin ended the entry, but nothing I read proves that: `goneEntryEnded` is not shown to have fired and `main-weft` is present on origin. Not verified: why the push path returned nil with nothing to push; whether the run's board entry for `records-push-main-weft-ended` is actually ended (I did not read the Bolt ledger, which would need `goneEntryEnded` logic and `ledgerRunsAt`). A judge should read the push function that calls `recordsSideNotLanded` and decide whether a nil push with an already-landed commit is a success path.
