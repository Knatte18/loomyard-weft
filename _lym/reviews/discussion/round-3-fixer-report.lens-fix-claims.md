```yaml
lens: fix-claims
outcome: reported
```

I checked the fix's added and rewritten claims against the code at HEAD.

Claims checked:
- **"Same blocking acquire as D10's lock, for at most 30 seconds"** (D3) does not hold against `internal/lock/lock.go`.
- **Leftover unload and wait-out guarantee** (D3) holds only partly.
- **Total-wait sentence** (D10 line ~211), left unchanged, is no longer true.
- **`StartupWMClass=konsole`** (D2 rationale) is confirmed in `/usr/share/applications/org.kde.konsole.desktop:12`.
- **KWin and KCoreAddons source facts** (Constraints) cannot be checked in this worktree. They are stated as read from upstream source and not observed, so nothing is claimed against local code.
- **The -1 load, the per-run plugin name and the end-marker timeout** follow from the stated KWin facts, with no conflict.

Findings:

1. MEDIUM — D3: "It takes the script lock through the same blocking acquire as D10's lock, for at most 30 seconds … a run that cannot take it in time fails."
   - Summary: `lock.AcquireWriteLock` cannot be bounded or abandoned while the child keeps running. D10's 120-second bound works only because the child exits with the acquire still pending. A script-lock run that "fails" and lets the child go on, to the next window-wait query or to settling and `window_command`, leaves a pending flock that can later take the lock and hold it until the process exits. The child's next acquire, on a new fd, then blocks on its own process's lock.
   - Evidence: `internal/lock/lock.go:30-40` says `AcquireWriteLock` acquires "blocking until available", calls `fl.Lock()` and takes no context. The only bounded acquire is `AcquireWriteLockWithin` (`lock.go:59-78`), which polls every 10 ms (`boundedWaitRetryDelay`, `lock.go:17`). The support log's "Rejected alternatives" turns down that poll for long waits.

2. MEDIUM — D3: "Holding the script lock until a run's script has left the list, and unloading a leftover first, keeps every lym run from reaching another lym script."
   - Summary: the guarantee breaks when the wait-out fails.
   - Evidence: the fix's own steps contradict it.
     - A run whose name still reads loaded after 2 seconds leaves it in `kwinscript.loaded`.
     - The next holder unloads it and "waits it out as below". If it still reads loaded, that holder then "writes its own plugin name there", which overwrites the leftover name.
     - The still-loaded lym script is then tracked nowhere. A later load can take its id once a script listed before it leaves.
   - The steps say nothing about a leftover that is still loaded when the bound runs out, such as refusing the load or keeping both names.

3. MEDIUM — D10 (unchanged sentence the fix reaches): "so a child that returns to the parent wait waits at most 10 seconds for its window, 180 for its parent and 120 for the lock in all."
   - Summary: under `kwinplace.lock`, each query and apply run can now wait up to 30 seconds for the script lock, plus up to 2 seconds of `isScriptLoaded` polling. None of that is in the stated total.
   - Evidence: D10 now says "Under the lock it runs the query, plan, apply and record steps, each script run taking D3's script lock inside it". D3 bounds each script-lock acquire at 30 seconds outside the window wait.

Further tool calls:
- grep of `internal/lock/*.go` function signatures: my context did not show which acquire functions exist.
- Read of `internal/lock/lock.go` lines 1-80: to check whether `AcquireWriteLock` can be bounded or cancelled, which the fix's 30-second script-lock bound depends on.
