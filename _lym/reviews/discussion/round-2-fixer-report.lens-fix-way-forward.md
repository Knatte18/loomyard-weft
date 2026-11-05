```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward: checked that each refusal, next step and row the fix added can be reached and makes progress.

No findings.

The new ways forward all hold:
- **D2 guard refusal (rename the Sketch):** `Board.Rename` refuses an entry that has a pair. Batten's start guard runs "before the pre-run writes anything", so no pair exists yet and the rename can be carried out. On a resume after step 3, the store holds this run's own Sketch with the same `created` stamp, so the guard passes.
- **D7 refusal (`raddle-approve` before a passing `raddle-propose` in the session):** the way forward is `raddle-propose`, which the agent can always run.
- **D9 steps 5 and 7:** step 5 runs after step 4 made the project a beam. Step 7 is the last write, so origin ending the run leaves nothing undone.
- **Halt between steps 3 and 7:** the entry is still open, so batten resumes the run and finds step 3 done by the store's hash.
- **D6 `board_checked` check and D8 bullet 2:** a failure is fixed when the agent adds the entry to `board_checked`. The cap of 2 returns bounds the loop.

Outside my lens: the D2 guard only compares the Sketch's slug against store directory names. A Sketch whose slug reuses a slug an older proposal *filed* still passes, and `lym board raddle <slug>` could then find two proposals.

Tool calls:
- grep of `internal/boardengine/rename*.go` and `runlock.go`: whether `lym board rename` refuses an entry with a pair. My context lacked the rename rules.
- grep for batten's start guard: to find where it runs. My context lacked it.
- `sed` of `internal/battencli/startguard.go` lines 1–40: whether the guard runs before the pair exists. My context lacked it.
