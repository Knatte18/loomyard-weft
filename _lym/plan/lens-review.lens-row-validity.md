```yaml
lens: row-validity
outcome: reported
```

**Review result (row-validity):** 1 BLOCKING, 7 MEDIUM and 5 LOW findings.

**BLOCKING**

1. **Cards 7 and 8: `TestDaemon_CrossHubDeliver` races the worker once card 8 lands.**
   - Card 7's rows post on the sending rig and send `deliver` by hand "since no worker runs until card 8". Card 8 then starts a worker for every pending post over `rig.hubs`, and lists neither `TestDaemon_CrossHubDeliver` nor `newHubPair` under `Edit`.
   - The failure: the hand `deliver` passes step 3 unheld, the worker stores and journals `remote-delivered`, and the hand `confirm` then answers `no`. That gives a spurious drop, a `CREEL` notice and a `refused`, so rows expecting `accepted` flake. Those are the second deliver, overlapping deliveries, the PanePID 0 row and the trust rows.
   - The rows that need a pending entry (retry after death, overlapping deliveries) no longer reach their branch.
   - Evidence: card 8 Intent ("started by `wakeOutbound`"), card 7 Intent ("no worker runs until card 8"), `Store.Confirm` step 5.
   - Fix: add those tests to card 8's `Edit`, and add a way to park the worker, such as a `Hubs` geometry error on the sending rig. Lookup and post would still work.

**MEDIUM**

2. **Card 7: the "confirmed and died before storing" row cannot use its mechanism.**
   - `FailJournalWrites` is a `Store` method (`internal/creelengine/export_test.go:21`), and `Run` builds its store internally (`internal/creelengine/server.go:78`). `rig` has no store handle, and `Options` has no fault hook on any card.
   - Fix: drive `Store.Deliver` on a hand-built `newEnv` store with `SetHubs`, or declare an `Options` test hook.

3. **Card 7: the `newHubPair` signature blocks rows on cards 7 and 8.**
   - It "starts both daemons" and returns only two rigs, so the `*running` from `rig.start` (`internal/creelengine/server_test.go:116`) is lost. Nothing can stop a daemon, and journal lines written after start are never folded (`foldJournal` runs only at `OpenStore`).
   - Rows this blocks:
     - Card 7: confirm after the remote-delivery and return records, whose lines must exist before start.
     - Card 8: receiver down, post while down, restart rows.
     - Card 8: every row that replaces the receiver with a scripted listener.
   - Card 8 lists `newHubPair` only under `Uses`.
   - Fix: return the running handles or start lazily, and put the helper under `Edit` where it changes.

4. **Card 8: "counted across a restart" would pass with the bug.**
   - `rig.start` resets both clocks to `epoch` (`internal/creelengine/server_test.go:118`). After a restart, a worker that wrongly counts its hour from the restart behaves the same as a correct one.
   - Fix: seed the pending `outbound` record with `rig.writeJournal` at `At` = `epoch` minus 40 minutes before start, and assert the return after 20 minutes. Alternatively, add a rig start that keeps the clock.

5. **Card 8: no scripted listener is declared, and `listenRingingFake` cannot serve.**
   - `ringingFake.serve` always writes a pending line and a ring line with `AckRequired` (`internal/creelengine/server_test.go:461-464`). It cannot answer `refused`, an unknown-verb refusal, an answer of neither word, or hold an answer open.
   - Rows that need a new helper: held `deliver`, neither-word answer, unknown-verb refusal, and the 200-byte quote.
   - The lost-`accepted` row states no mechanism at all. One that works is a silent listener until the timeout, then a real daemon started over a journal holding the inbound record.
   - The helper needs a `Create` handle. Rigs also need the stop handle from finding 3.
   - `Hubs.Add` twice answers D1's "several" refusal, so the fake cannot repoint a socket either.

6. **Card 7: the overlap and mutual-delivery rows do not fail if the code is wrong.**
   - Two concurrent `rawExchange` calls usually serialize, so the second hits step 3, and the step 6 re-check is almost never reached.
   - "Two hubs delivering to each other at once" passes unless the timing lines up. A held store lock would then hang on a fake 10-second timer rather than fail.
   - Fix: script the sending socket so it holds `confirm` until both calls arrive. For the lock rule, assert that a verb on the receiving store answers during a held confirm.

7. **Card 7: branches with no row.**
   - A failed or timed-out confirm (D3: transient, no drop, nothing stored). This is the negative that keeps a sender that is down from counting as forged.
   - A confirm answer of neither word.
   - A failed strand listing.
   - A `deliver` with no `Mail`.
   - Also wrong: the nil-seam refusal is called "a fail-loud wiring guard no entry point reaches in production". Card 4's `serveDaemon` leaves `Hubs` nil when `registryDir` fails, and a hand-written `deliver` or another hub's worker then reaches it. `rawExchange` on a nil-`hubs` rig reaches it in a test too. It needs a reaching row, per the rubric's "Reachability of a prescribed refusal".

8. **Cards 6 and 8: the sweep rows have no working observation, and card 6's new `Store.WaitReply` and `settledOutcomeLocked` branches have no row.**
   - With no sweep subject the sweep arms no timer, so `sweepStep` hits `AwaitWaitsWhere`'s 10-second give-up and then indexes an empty slice (`internal/creelengine/server_test.go:318-319`). The rows that do this are card 6's `TestSweep_RemovedHolders` row and card 8's "not settled by a sweep".
   - After card 8, the 30-second call timer and the retry waits also pass `isSweepWait`, `isProbeWait` and `isChaseWait`, so those predicates no longer tell the loops apart. Either state each row's observation, or use `Advance` with a log assertion.
   - Unreached branches:
     - `WaitReply` on an outbound id (the `--wait-reply` reconnect path).
     - A returned entry answered at wait registration.
     - An already stored reply answered at wait registration.
   - Rows exist only for `PostWait` and live waits.

**LOW**

9. **Card 4: unstated `creelfake.Hubs` zero values.** Card 4 does not say what these answer:
   - `Spawn` with no `SetSpawn`
   - `Geometry` for an unknown path
   - `TrustedHubs` before `SetTrusted`

   Card 6's sweep row, once card 8 runs a worker over it, depends on the first two.

10. **Card 9: the end-to-end `TestCreelCLI` row needs changes no card lists.** `rig.startDaemon` passes no `Hubs` (`internal/creelcli/cli_test.go:311`), and `newRig` builds one `tst` hub and its geometry with no shortname option (`cli_test.go:285-290`, `internal/hubforge/hub.go:70`). `internal/creelcli#rig.startDaemon` and `newRig` belong under `Edit`.

11. **Card 4: two `TestHubDirectory` groups repeat other tests.** The absent and invalid base-file rows repeat `TestLoadConfig`, and Record-then-Lookup and the unknown-shortname row repeat `TestLookup`. Only the no-restart row and the `Geometry` row are new at this surface.

12. **Card 7: one clash row repeats another.** The "all but the post time" row takes the same branch as the post-time variant of the differing-fields row. Fold them into one row, naming both criteria.

13. **Card 1: "two goroutines recording" rarely fails without the lock.** Use many goroutines with one hub each.

**Tool calls beyond context**

- Read `internal/testkit/creelfake/creelfake.go`: the fake clock and `AwaitWaitsWhere` semantics were not in context.
- Ran one grep batch: other-shortname test names, `clientSeam` literals, `UpdateJSON`, tmuxkit `setTempEnv`, `XDG_STATE_HOME` tests, the hubforge shortname, the `TestModules_LocalLayerRefusals` table and test tiers. None of these were in context.
- Read `internal/creelengine/server_test.go:1425-1484`: `listenRingingFake` lay past the part already read.
