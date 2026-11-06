```yaml
lens: guard-reach
outcome: reported
```

Guard-reach review of the plan: for each guard, refusal, branch and test row, can its state occur, and does the guarded site recur unguarded.

**BLOCKING: card 7 step 3 / card 3 — a genuine retry reads as an id clash once the message has moved**
- `heldLocked` compares the delivery's receiver with a held message. `Message` keeps only the current `Holder` (`internal/creelengine/fold.go:38`), and card 3 adds no original-receiver field.
- `Store.Forward`, and a sweep's `eventReturned`, change `Holder` (`fold.go:267-282`).
- Reachable sequence:
  1. The receiver stores the message and its `accepted` is lost.
  2. The holder forwards it locally, as D4 allows.
  3. The worker retries, or makes its final call at the hour.
  4. The comparison differs, so the answer is `refused` with a clash.
- The sender then journals a return for a message the receiver holds. D3 step 3 and D2 require `accepted` with the held id, and no row covers it.
- Fix: journal and fold the original receiver on the inbound message, or compare against `Path[0]`, and add a row.

**MEDIUM: card 7 steps 4–5 — the body-hash guard is bypassed if the caller's hash is trusted**
- `HubMail` carries both body and body hash, and nowhere says the receiver computes the hash from `Body`.
- A hand-written `deliver` that replays a pending tuple with its true hash and an altered body gets `yes` if the field is forwarded. The altered body is then stored under a confirmed sender.
- The planned row "a differing body hash" tests only `Confirm`.
- Fix: state that `Deliver` ignores a supplied hash and computes it for steps 3, 4 and 6, and add this forged row.

**MEDIUM: card 7 step 2 — every `Lookup` error is treated as a permanent refusal**
- D3 step 2 refuses only on none or several. Card 1 also makes `Lookup` refuse a corrupt record and can propagate read errors.
- `HubDirectory.Lookup` returns a plain error, so card 7's "a refusal answers `refused`" returns a genuine message at once on a damaged `hubs.json`.
- Fix: give none and several a typed answer, and let any other error take the transient path. Add a row with a `Hubs` lookup error.

**MEDIUM: card 7 — the "receiver died before storing" row cannot reach its branch**
- The row uses the journal write fault over `newHubPair`. `FailJournalWrites` is a `Store` method (`internal/creelengine/export_test.go:21`).
- `Run` builds its store internally (`internal/creelengine/server.go:78`), so a rig holds no `Store`. No `Options` hook or `export_test.go` edit is listed.
- Fix: list the seam, or drive the row some other way.

**MEDIUM: card 7 step 6 — no deterministic row reaches the second duplicate check or its clash branch**
- "Two overlapping deliveries" over `rawExchange` are only concurrent. Both must pass step 3 before either stores, and nothing holds them between the confirm and the lock.
- The step 6 clash branch has no row at all.
- Fix: script the confirm answer with a held listener (the `listenRingingFake` shape, `server_test.go:1433`) and list it.

**MEDIUM: cards 7 and 8 — rows need journal lines and stops that `newHubPair` does not allow**
- `newHubPair` "starts both daemons" and returns only `(sending, receiving *rig)`.
- `rig.writeJournal` only takes effect before a start, since the store folds at open (`internal/creelengine/pool.go:108`).
- Stopping needs the `running` handle `rig.start` returns (`server_test.go:116`). `rig` is under `Uses` on card 7.
- So the card 7 rows for a confirm after remote delivery or return have no way to get their records folded. The card 8 rows for a receiver that is down or a sending rig that restarts have no way to stop a daemon.

**MEDIUM: cards 6 and 8 — the card 6 sweep row and the card 8 sweep row collide with the worker's timers**
- `isSweepWait` is `8s < d <= 50s` (`internal/creelengine/deliver_test.go:29`). The worker's 16s and 32s retries, the 30s `deliver` timeout and the client's timer all fall in that window.
- `env.sweepStep` picks the last such wait and awaits one more, so it advances the wrong timer or hangs.
- Once card 8 lands, the card 6 `TestSweep_RemovedHolders` row runs a worker under its scheduler. The card 8 row "not settled by a sweep" has the same problem.
- No card updates the helpers or the row.

**LOW: cards 4, 7 and 8 — the nil-seam guard is reachable in production, and its site recurs unguarded**
- Card 7 says the nil-seam refusal is a wiring guard "no entry point reaches in production". Card 4 leaves `Hubs` nil in `serveDaemon` when `registryDir` fails, so a `deliver` reaches it. It then needs a way forward and a test.
- Card 9's `Store.Reply` has no seam guard, and card 8 starts no worker on a nil seam. Under a nil seam a reply, or an already pending outbound message after a restart, stays pending with no hour return, against D2's "also across daemon restarts".

**LOW: card 9 — `TestCreelCLI`'s end-to-end row cannot reach `hubDirectory`**
- `rig.startDaemon` builds `Options` with no `Hubs` (`internal/creelcli/cli_test.go:311`), and no card lists it. Card 4 edits only `newRig`.

**LOW: card 7 step 6 — a `no` after an overlapping delivery drops a genuine message**
- A timed-out attempt still in the receiver can get `no` once the retry's `accepted` is journaled, and it journals a drop and notifies the orch for a stored message.
- The record's step 6 rechecks only on `yes`. A recheck on `no` would avoid a false drop, which the operator's added decision treats as a lym weakness.

**Guards with no finding:** every other refusal and branch on cards 1, 2, 4, 5, 6, 8 and 9 traced as reachable, apart from those listed above. `internal/state#UpdateJSON` exists (`internal/state/state.go:159`). Only one test names a foreign shortname, the `zz:orch` post row (`internal/creelengine/verbs_test.go:215`).

Tool calls: one grep of `internal/state`, the creel test files and `creelfake`, because my context lacked `UpdateJSON`, the foreign-name test rows, the clock API and how `FailJournalWrites` is used.
