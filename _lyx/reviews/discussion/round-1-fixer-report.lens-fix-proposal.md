```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged the proposals the fixer carried out and found 4 problems.

1. **MEDIUM — D4, "batten's wiring of the Go driver's `PauseHold` ends the pass as a yield does, exit 0 with the run `running` at its row"** (from F4).
   - **Summary:** The driver calls its pause hold only at a stop it has already met. Ending the pass there leaves the run in its stop state, not `running` at its row. Dobby passes only `running` runs, so after the lift no pass picks the stop up again. Today the driver waits and then goes on to the tackler or arbiter.
   - **Evidence:** `holdPause` is called only inside stop handling in `internal/sheddriver/drive.go`:
     - after a rule decision, before `recheck` (around line 251);
     - in `handOver` (line 315);
     - in `tackle` (line 418);
     - in `arbitrate` (line 509).
   - D4 says "Dobby runs a pass of a run it drives only while batten's own state is `running`". D12 says the lift pass reaches only "every running run it covered". So a halted stop under a pause waits for a hand pass, and the tackler and arbiter are skipped.

2. **MEDIUM — D4, "`pairteardown`'s liveness read itself (`battenAlive`) reads a run as having a live lifecycle owner when a registered batten lives or a live dobby drives it, so every reader of it … takes it"** (from F9).
   - **Summary:** "Drives" covers halted runs too, because D4 says dobby drives them but passes them only while `running`. With this rule, `lyx reed down` of a pair whose run halted is refused while its parent's dobby lives, and the refusal names a pause for a run that is not running.
   - **Evidence:** `settledForSessionDown` in `internal/pairteardown/skip.go` returns `false` on `alive` before it reads `landed` or the quiet state. `EndBeforeSessionDown` then refuses with `ErrRunRunning`: "pause it with `lyx tender pause`". Today a halted run reads not alive and not running, so it is settled.
   - `driverChecksSkipped` likewise stops skipping the quiet wait for a run that halted after Finalize Done.
   - The dobby arm should count only a run dobby passes: `running`, with no cancel marker.

3. **MEDIUM — D4 respawn cap, "a pass that sees a live driver strand or a held run or driver lock resets the count"** (from F8).
   - **Summary:** A driver that crashes each time but outlives one probe interval resets the cap after every respawn. It is then respawned forever and the driver-dead notice is never sent.
   - **Case:** A pass respawns the driver. The driver holds its lock or a live strand for a few minutes and then dies. The next probe pass sees it live and resets the count. The pass after that finds it dead and respawns it, and the loop repeats.
   - The old cap of three per child state episode caught this case. A reset needs proof of progress, such as a write of the child's status file since the respawn, not mere liveness.

4. **LOW — D10, "Dobby counts a landing from the batten history of every run it drives, read on each tick"** (from F11).
   - **Summary:** Run records outlive teardown, so a dobby's first start, or a lost state record, counts every old Land Done in the hub's batten runs directory as a new landing. That spawns a tidy for stale slugs.
   - **Evidence:**
     - D3 finds runs "from the local batten status files of the hub".
     - The run directory is removed only by `os.RemoveAll(shedrun.RunDir(location, slug))` in `internal/battencli/cleanup.go:277`. Teardown (`internal/battenshed/teardown.go`) removes only single paths.
     - The old "end of a pass" rule could not count a landing from before the dobby existed. The new rule needs a baseline taken when the dobby first starts.

Tool calls:
- Read the Go driver's `holdPause` call sites in `internal/sheddriver/drive.go`, to see where the pause hold sits. My context had only the `holdPause` body and its line numbers.
- Grepped the batten docs for whether status files outlive teardown. No hits.
- Grepped `os.Remove` and `RemoveAll` in `battencli` and `battenshed/teardown.go`, to find what removes a run's records. My context did not show it.
