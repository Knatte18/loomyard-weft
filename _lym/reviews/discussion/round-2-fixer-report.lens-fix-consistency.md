```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-2 changes agree with the parts of the decision record and support log they touch or depend on.

1. **MEDIUM.** D7 step 4 does not match the new acceptance criterion "a `StopStrand` during a relaunch leaves the run stopped and no strand up".
   - Step 4 re-reads the record for a terminal outcome only after reed has relaunched the pane.
   - A `StopStrand` during the window runs `removeStrandIfPresent` on the held strand. Under `strandRemovable` (`internal/shuttleengine/stop.go:135-137`), a held own-window strand with its pane present is removed.
   - Step 4's same-GUID relaunch then has no strand to act on and fails.
   - The changed text sends that failure down "Any other refusal, or a failure in steps 2 to 4, ends the run as a strand that died". That path never says it leaves a terminal record alone, so the stopped record can be overwritten with died.
   - Fix: step 4's re-read (or the failure path) should keep a record that is already terminal.

2. **LOW.** The new orphan-sweep clause has two versions that disagree.
   - D7 step 1 says, with no end: "The orphan sweep keeps a run directory whose record reads relaunching."
   - The PATTERN changes bullet says the sweep keeps it "until the mark clears or its bound passes".
   - Past the bound, with no attach or `Wait` to end the run, step 1 keeps the directory for good, while the PATTERN bullet says the exemption has ended.

3. **LOW.** The new reed ops are named in the Constraints bullet and the plan-writer note, but nowhere in Scope or in the seam's interface.
   - The ops are: end a strand's process while keeping it held, and relaunch its pane under the same GUID with a new `Cmd` and `ResumeCmd`.
   - Shuttle reaches reed through `ReedOps` (`internal/shuttleengine/reed.go:25-36`), which has neither op.
   - The Scope "In" list names no change to `internal/reedengine`. The only twin of the new Constraints wording is the Notes bullet.

No further tool calls made.
