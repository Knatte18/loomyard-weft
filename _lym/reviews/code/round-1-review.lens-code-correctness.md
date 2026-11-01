```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff, walking each function and branch with a concrete input.

**List (function or branch: case run: verdict)**
- `EntryStamp` (`record.go:43`): history of Preflight Done, Record-Write Done, then a halt of the row gives stamp 1 and stays at the row. Correct.
- `EntryStamp`, resume: a resume appends no entry, so the stamp stays 1. Correct.
- `EntryStamp`, Done of the row, then Finalize reroute `goto` onto Code-Bouncer, then Bouncer Done, Describe and later rows: the later `goto` back to the row raises the stamp only when `atRow` is false. Correct.
- `samplerStamp` (`samplerwiring.go:32`): an absent status file or a run directory not yet created gives 0. Correct. The 0 can't match a flag, because `flagIsCurrent` ignores `DoneStamp == 0`.
- `Call` (`samplerSession.go:91`): a `*samplerHalt` from `ensureSession` inside `judgeDone`, `refuseDirtyDone` or `wake` is converted by `errors.As` in `Call` into Stuck with Halt. Correct.
- `Call`/`endSession`: Stuck, Done and non-cancel errors stop a live strand. A cancelled context leaves it running. Correct.
- `execute`, first entry (empty record): `flagIsCurrent` is false, `ensureSession` starts with an empty resume id, the record is written, and the wait begins. Correct.
- `execute`, current flag with a clean checkout and intact record: `judgeDone` stops the live strand, never starts one, and returns Done. Correct.
- `execute`, current flag with a dirty checkout: the strand is stopped, the flag cleared, `ensureSession` resumes by the recorded id, `rowStopped` goes back to false, the mail goes to the new `StrandName`, and the call waits. Correct.
- `execute`, current flag with a damaged record: Stuck with Halt without starting a session, also when the session can't be resumed. Correct.
- `ensureSession`, record write failing after a start: the new strand is stopped and the error returned. Correct.
- `ensureSession`, transient start failure: returns an error the driver re-steps. Non-transient: halt. Cancelled context: the cancel error comes first. Correct.
- `wake`, covered: stops the session once and judges nothing else. Correct.
- `wake`, first uncovered wake after a covered one: `ensureSession` runs before the flag check, which gives the resume you want. Correct. If the flag is current, `judgeDone` then stops it again, which is fine.
- `wake`, dead strand: Stuck with Halt only when `!rowStopped`. After a covered stop `rowStopped` is true, and `ensureSession` clears it on resume. Correct.
- `wake`, unreadable pause record: reads as uncovered and logs a warning. Correct.
- `pause` and `nextSamplerWait`: the interval sequence is 1, 2, 4, …, 60 s and resets to 1 s on a change. A `Watch` event cancels the sleep, and the remainder of the 1 s floor since the last wake is still waited out. Correct. The watch is on the record's directory, filtered to the record's basename (`landingdeps.go:376`), so the lock file does not retrigger it.
- `samplerDoneVerb` (`samplerdone.go:50`): all four refusals (no row, other row, no session recorded, dirty) leave the record unchanged. The success write uses `samplerStamp`. A halted run at the row is accepted. Correct.
- `samplerStrandReader` (`decision.go:944`): a darn run gives false with no reads. A missing status file gives false (`circlingStatusReader` handles not-exist). A status at another row gives false. A sampler run's row name with `--by operator` is admitted. Correct.
- `decisionVerb` caller rule: the sampler strand with `--by operator` passes, any other label is refused with `decisionWaySampler`, and any other strand falls through to the parent-orch rule. Correct apart from F1 below.
- `codeBudgetZero` and `rerouteReviewed` (`landingdeps.go`): a soft budget of 0 gives off. The darn and tapestry Bouncers carry no gate, so `CodePerchGate` returns nil and the result is unchanged. A sampler run with `code_review` false gives off, and the Finalize reroute halts. Correct.
- `wire`, `wireTreeScope` and `wireDarn`: darn declares `verify_source: tree_scope` (`darn-recipe.yaml:64`), so darn still gets the tree-scope seams. `SegmentBounces` is now set only in the general Env at `wiring.go:813`, which uses the same `loomrecipe.Routing(c.recipe, loomSegmentBudgets(loomCfg))` as the removed darn-only assignment. Correct.
- `runWaitsOnOperator` (`looparm.go`): an absent status gives false. An unreadable status logs at debug and gives false. Running at a listed row gives true. Correct.
- `childAtWaitingRow`: `loomshed.WaitsOnOperator(CurrentProducer)` with the `running` check is unchanged in meaning. Correct.
- `SessionSpec` (`spec.go:43`): an empty required field is refused by name, and an unresolvable agent is refused. Correct.
- `LoadConfig` (`config.go:52`): an invalid `agent` is refused naming the key. Correct.

**Findings**

F1 — LOW, design, `internal/loomcli/decision.go:651-654` (evidence: the lines added in the `decisionVerb` caller rule).
- A named caller is checked by `samplerStrand` before the parent-orch comparison, and a read error from it refuses the call.
- On a sampler run at Sampler-Session, an unreadable sampler record or status therefore also refuses the run's parent orch with "read the run's sampler session".
- Before this diff the parent's call did not depend on that file.
- Fix: on a `samplerStrand` error, fall through to the existing parent check instead of refusing, and refuse only when the caller is not the parent.

No other defect found in this lens.

**Tool calls (1)**
- One Bash call read `wiring.go:805-822`, the `verify_source` lines of the recipes, `circlingStatusReader` and `watchFileChanges`. My context lacked the general `SegmentBounces` wiring, whether darn declares `tree_scope`, and the not-exist and file-filter behavior of the status reader and the watch.
