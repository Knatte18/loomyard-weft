```yaml
lens: guard-reach
outcome: reported
```

Task: guard-reach lens review of the Plan cards, checking whether each guard, refusal or branch they add can be reached from a real entry point.

**Findings**

1. MEDIUM — Card 8 (`Command` RunE, D1/D2/D7). The `RunE` dispatch branches have no planned test that reaches them through the verb. They are the `--discard`-with-subject-or-flag refusal, follow-and-spawn-nothing, the caller's other-subjects refusal, and `NoOrch` set from an orch strand. `spawnWorker` refuses under a test binary, and RunE gets no seam, so every planned row stops at the helpers (`TestDiscard`, `TestRunsInOrch`, `TestFollow`, `TestJudgeStanding`). Two criteria are reached at helper level only. One is D2 "a re-run while a live worker runs follows it and spawns nothing". The other is D7 "from an orch strand… starts the dobby and no orch". Evidence: `internal/adoptcli/deps.go:237-240` (`startSuite`'s `testing.Testing()` refusal, which the card copies for `spawnWorker`). Card 8's test list has no RunE row.

2. MEDIUM — Card 6 `stepPairs`/`stepDone` (D3 held-back). `Inputs.HeldBack` is defined as "the projects a local-commit stop held back". Today's `held[project]` is also set when `takeOverSlug` returns false on an error such as a refused pair add (`internal/adoptcli/adopt.go:326-328`, `out.Error` set in `takeOverSlug` at `adopt.go:806-808`). If `stepPairs` fills `HeldBack` from that map, `stepDone` ends a pair-add refusal `stopped`, naming `--keep-local`/`--drop-local`. That way forward is wrong: the add's own way forward is the archive tag (`addError`, `adopt.go:820-826`). The card does not say to exclude error holds.

3. MEDIUM — Card 6 `stepPause`. "A failed pause update stops the record at `pause`, naming the re-run" has no planned reaching row. The fake `world.deps().UpdatePause` cannot fail (`internal/adoptcli/adopt_test.go:196-199`), and the card lists no new world field to make it fail.

4. LOW — Card 7 `runWorker` step 3. The branch that treats a `follow` verdict as a refusal naming `lym beam status` cannot be reached. The worker reaches step 3 only while holding `adopt.lock`. A live worker with matching start ticks always holds that lock, since a worker that loses it "exits at once and writes nothing" (D2). So the state the branch guards cannot occur, and it needs either a fail-loud wiring-guard label or removal. The planned `TestJudgeStanding` follow row covers only the pure function.

5. LOW — Card 6 `pauseEntries` (D5 beam rule). The rule says "a beam slug gets a conversion of its own-scope `OwnerCaller` entry and nothing else; a beam with none gets no entry". That collides with the adopted beam's pawl run being a pause target. `movedRuns` lists an adopted beam that holds a run record (`adopt.go:679-688`), and `newWorld`'s `alpha` and `beta` hold running records (`adopt_test.go:109`). The no-entry guard is therefore reachable only for a beam with no live run, and the card does not say the target rule wins for a beam that is in both `slugs` and `beams`.

6. LOW — Card 8 `follow` under the follow verdict. When the caller spawns nothing, the card does not say what `follow` gets for `exited` or `workerPID`. The lost-lock branch ("once `exited` is closed…") is then unreachable on that path. The card should state a nil channel and the record's pid.

7. LOW — Cards 3 and 8, outside this lens (way-forward). `adoptrecord.Read` refuses a record that does not decode, naming `lym adopt --discard`. But `discard` removes only "an unfinished record", and judging that needs the decode, so the named next step cannot get past the same bad file.

Tool calls:
- grep plus a read of `internal/battencli/cancel.go`, `cancelcmd.go` and `cancelcmd_test.go:110-140`. My context lacked whether `lym batten cancel` lifts own-scope entries of any caller. It does, so an `AdoptCaller` entry is covered there and nothing is flagged.
