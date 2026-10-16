---
verdict: APPROVED
---

No findings.

The range is `git diff $(git merge-base main HEAD)..HEAD`, the eleven card commits: `_lyx/shed/creel-stage1-fixes/status.json` names `main` as `product.parent` and no `_lyx/loom/rework` round exists.
`lyx gate test ./internal/creelengine ./internal/creelcli ./internal/shuttleengine` and `lyx gate test --tags integration ./internal/creelcli` pass.

Per focus directive, each point was read against its card's Intent in the code path and in the tests:

- Cards 3 and 9: `freshStallLocked` drops a stall only when `Sub > stallFreshness` (16 s), so a reading exactly at the limit is fresh, and `noteStall` uses `<=` the same way. The probe-wait cap is `min(next, max(remaining, probeBackoffBase))`, applied only when `w.running`, and it leaves `lastWait` uncapped so the backoff state is unchanged.
- Card 6: the first miss sets the 10 s confirm, the ceiling is 50 s, a wake during a pending confirm re-arms at the time left to the deadline, and a wake outside a confirm re-arms only when more than the base is left. New mail never lengthens a wait. The confirm-ending sweep doubles from the base to 50 s, and a new first miss at that sweep sets a new confirm.
- Cards 7 and 8: `NoInputBox` is set on the readiness path only for the no-input-box reason, and on the pane path only when the engine reads a box, found none and the pane is tall enough. `hubDelivery.Idle` ranks it after too-short and draft, and a busy or absent pane names no stall.
- Cards 10 and 11: `Runner.AcceptStartupGate` calls only `Engine` methods, so no provider wording leaves the engine adapters. `TestProviderSeamImportRule` pins the single caller outside the package. The smoke test passes `--anchor own-window`, polls until the gate is accepted or the box shows, and prints the pane on both failures.
- Cards 1, 2, 4 and 5: `repairTail` and `foldJournal` share `lastNonBlankLine` and `parseRecord`, so the cut line is the skipped line. `writeRefusal` falls back to stderr on a stdout write failure and keeps the exit code. `connect` ends the wait on a non-race exit after one last dial and keeps the backoff on a lost race. Carried records fold through `apply` after a fold and in `retain`.
- Docs: the `doc.go` paragraphs, `pattern/PATTERN-daemon-wakeups.md` and the C4, C6, C8, C10 and C11 lines of `tools/sandbox/SANDBOX-CREEL-SUITE.md` match each card's behavior.
- Test economy: each new row pins a behavior its card names, and I found no duplicate of an existing test's coverage.
