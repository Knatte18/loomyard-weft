```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, run for each function or branch the diff adds or changes.

**Per function: case run, verdict**
- `runForeground` (foreground.go:57): killed-adopt re-run, own hold over the same set, `Kept` true. It writes only the owner and registry entry, then `holdStands` passes. Correct.
- `runForeground`, check end (foreground.go:177-201): check green, check failed with a task, and check failed before any tier (`FailedTier` nil, so no task). The last one ends with an error and the hold standing, and `runBeamSteps` still runs. That matches D6's "BeamUp on red". Correct.
- `holdStands` (foreground.go:230): own hold over the same set returns nil. Another machine's hold gives a stop naming that machine. A cleared hold gives a stop naming the owner. When the owner lookup errors, `owner = "unknown"` and the stop still fires. Correct.
- `movedRun` (foreground.go:275): five inputs. An entry naming this machine is not moved. An entry naming another machine is moved. No entry with the prior owner here is not moved. No entry with the prior owner elsewhere is moved. The fix task filed by this machine is not moved. This matches D6 step 2. Correct.
- `takeOverRuns` (foreground.go:301): entry left nil, `AddPair` failing, `TakeOver` failing. Each failure stops naming the run and the re-run. Correct.
- `planAdoptHold` (holdwrite.go:209): another machine's hold without `TakeHold` refuses with `otherMachineRefusal`. With `TakeHold` and the same set it rewrites the hold and keeps kind, fix task, tries and prior owners. With `TakeHold` and another set it refuses naming that hold's subjects. An own hold over the same set is kept. An own hold over another set or an overlapping one refuses with the re-run `lym adopt <held> first`. Correct.
- `WriteAdopt` (holdwrite.go:292): rejected push, then owner re-read. Replay refuses whole on another machine's new hold, and the post-push read returns `otherMachineRefusal`. A lost owner write returns the `OwnedElsewhere` refusal. Correct.
- `FixHold` (holdwrite.go:385): another machine's hold, no hold, and an identical hold (a no-op). One defect, F1 below. Otherwise correct.
- `ClearHold` (holdwrite.go:449): no hold, another machine's hold, and a replay that lost the removal. Each returns the right result. Correct.
- `refuseOverAnotherMachinesNewHold` and `changeHolds` (replay.go): a hold rewrite, a clear, an append, and a hold the origin no longer has. The positional `replaced` test handles delete-at-0 and replace. Correct.
- `changeRegistry` (replay.go): an entry added or changed here versus one the origin changed since the base. Correct.
- `runHoldCheck` and `endGreen` (holdcheck.go): no hold, another machine's hold, `first` outside the hold, a lock held (`Ran` false), a hold lost at the end, a failed `ClearHold`, and a failed bolt-hold drop. Every one gives a way forward and no stale state. Correct.
- `endFailed` (holdcheck.go:257): `FailedTier` nil files no task. A `lost` refusal with `OwnStands` false brings nothing up. Correct.
- `TakeOverRun` and `land` (takeoverwrite.go): a held teardown lock, a push rejected then retried, retries exhausted, and a non-rejection push error. Each drops the entry commit or returns the error. Correct, apart from F2 below.
- `DropHandOverMark` (handover.go:143): an absent mark returns nil. Correct.
- `hereRequest` (here.go): the record branch is removed, with one Bolt pull and `owned_here`. Correct.
- Shell-guard row (rows.go): `lym adopt --take-hold` and `lym adopt a b --take-hold` match. The steer has no quote and no flag spelling. Correct.

**Findings**

F1 — MEDIUM, design
- Location: internal/boardengine/holdwrite.go:386 and 403, 439; fed by internal/adoptcli/fixloop.go:411.
- Summary: the `FixHold` way forward names only the subjects in the `tries` map, which `fixloop.go:411` passes as `{subject: tries}`, the failing subject alone.
- Evidence: for a multi-subject hold, such as `lym adopt a b`, with `b` failing, a lost hold or a missing hold yields `run lym adopt b`. D6 refuses any set other than the hold's, and `planAdoptHold` (holdwrite.go:237-243) refuses it as an other set, so the way forward does not work.
- Fix: take the hold's own subjects, from `want.Subjects` or the own hold read in the write closure, when building the way forward at holdwrite.go:403 and 439.

F2 — LOW, design
- Location: internal/fabricengine/takeoverwrite.go:264-266.
- Summary: `if err != nil || !hasUpstream { return ...%w, err ...}` wraps a nil `err` when there is no upstream.
- Evidence: the message prints `%!w(<nil>)` and does not say that the branch has no upstream. The retry path then fails with a misleading error.
- Fix: return a separate error for `!hasUpstream` naming the missing upstream of `b.weft`.

**Tool calls (1 of 6)**
- `grep` of `FixHold` and `Tries` in fixloop.go, and `movedRuns` in adopt.go: my context lacked the argument `FixHold` receives and what `OriginRun` returns.
