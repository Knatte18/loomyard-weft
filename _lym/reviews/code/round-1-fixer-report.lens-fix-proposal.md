```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals of F2–F7 and F9 that the fixer carried out as proposed.

No finding rises above LOW.

1. **LOW — F5, `claim` via `os.Link`.**
   - The change is the temporary file plus `os.Link` to `process.json`.
   - It breaks on a filesystem without hard links, such as some bind mounts, network shares and FAT.
   - `os.Link` then returns an error that is not `os.ErrExist`, so `runDetached` refuses with "cannot be claimed", and `awaitClaim` gets a non-`ErrExist` result from `claimByFront`. Every capped call on such a filesystem ends in `failStart`.
   - The old exclusive create worked there.
   - The review's second option, a retry in `readJoinState` of a present but undecodable process file, has no such dependency.
   - `os.CreateTemp` also makes the file 0600, so `process.json` loses the 0644 it had. The readers are the same user's processes, so this is cosmetic.
   - A crash between create and remove leaves a `process.json.*.tmp`. That is harmless, because pruning removes the whole directory.

2. **LOW — F7, verb-aware text is incomplete.**
   - Only `replayedBusyLine`, `abandonedLine` and `answerRunGoing` take the verb now.
   - Sibling front messages in `detachfront.go` still hardcode `gate test:` for a card-verify call: the `output.Err` strings in `run`, `lockKey` and `startRun`, `failStart`'s "gate test: the detached runner did not start", and the `logger.Warn` lines.
   - A card-verify caller therefore sees a mix of `gate test` and `gate card-verify` in one session.
   - The review named only three sites, so this is a leftover twin rather than a missed finding.

3. **LOW — F2, a partial copy is replayed twice.**
   - When stdout copies and stderr fails, or the writer fails midway, the caller already received part of the output.
   - The run is not marked delivered, so the next call replays the full output again, and the caller sees duplicated stdout.
   - This is the trade-off the review chose, since a lost result costs a whole new gate run. If the writer keeps failing, such as a closed pipe, every call repeats the replay until the lease lapses.

4. **LOW — F3, a permanently unreadable lease.**
   - `leaseLapsed` answers false on a read error, so a lease file that stays corrupt never cancels the runner.
   - The runner then lives until its call ends on its own. A hung tier would be a leaked runner with no lease bound.
   - The old behaviour was the opposite failure. The fix still gives up the lease's upper bound for a corrupt file.

Checked and found sound:
- **F4:** I built the race (watcher stores `passed`, main reads `callCtx.Err()` before `cancel()`), and it yields the real verdict, which is correct.
- **F6:** pruning runs under the key lock from `startRun`, skips a live runner, and skips the `lock` file as a non-directory.
- **F9:** the `keep_warm_cap` keys and the 250s value match `claude.yaml`.

Further tool calls: none, since my context held everything needed.
