```yaml
lens: code-scope
outcome: reported
```

Lens code-scope: I traced each change in the diff to the decision it builds.

**Decisions** (a decision record alone; the goal points come from the board entry's `## Goal`, plus the one Added entry):
- G1, Seen: the named subtest `TestStore_MachineOutbound/WithTheSchedulerStartedAndNoTake…BridgeUnsettled` fails at times and passes on a rerun. The failure is the test's own. Built by C1.
- G2, Idea: find what the subtest waits on and make it pass every run. The Idea suggested waiting on a signal instead of on time. Built by C1, in the form A1 settles.
- A1, Added after Discussion (parent, 2026-11-28), "No test seam": the fix is test-only. It posts both messages before the scheduler starts, so the loop's first pass sees both and arms the single hour wait. It needs no red-first check and changes no production code. Built by C1 and C2.

**Changes** (one hunk, `internal/creelengine/machinemail_test.go:144-156`):
- C1: `e.start(t)` moves from before the `PostWait`, `ReportMachineHold` and `e.post` calls to after them. It traces to A1, and through A1 to G1 and G2.
- C2: a two-line comment above the posts says why they come before the start. It traces to A1, as the comment C1's non-obvious ordering needs.

**Coverage**
- Every change traces to a decision, and every decision is built. The diff holds no production change, which matches A1's "test-only".
- The commit message for f84f1a937 carries no `Pattern-Departure:` trailer, and none is needed.

**Findings:** none.

**Tool calls:** none.
