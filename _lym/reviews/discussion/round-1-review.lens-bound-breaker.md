```yaml
lens: bound-breaker
outcome: reported
```

Claims attacked (each held or broken):

1. D3 "started detached in a new session … outside every pane's process tree … creel reads it as no strand" — **broken**.
2. D3/D6 `lym slack away` waits for the relay's first read, which makes `SLACK` known — **broken** (by 1).
3. D3 `lym slack stop` ends it through `EndRole` — **broken** (undone while presence is `slack`).
4. D3 registered under `runproc.HubOwner` with a role of its own, ended by `EndRole` — held.
5. D3 re-exec like the hub watchdog with `proc.ReapInheritedChildren` — held.
6. D3 first read makes `<shortname>:SLACK` a known receiver — held.
7. D4 4 KiB cap = the size a pane-mail orch still gets typed in full — held.
8. D4 pause: post stored, delivered at resume — held.
9. D4 bound "a hold … delays the mail up to the hold's hour" — **broken** (renewal).
10. D4 bound "a crash between the creel post and the cursor write forwards once more" — held.
11. D4 rationale / Constraints "stays inside the history method's limit … across the machine's hubs" — **broken** past four hubs.
12. D5 "a reply to a `SLACK` message reaches the relay's mailbox" — held.
13. D5 release-unacknowledged/repost — held (at-least-once).
14. D5 bound "creel's seven-day retention bounds how long mail waits" — **broken**.
15. D5/D7 only orchs and programs reach the channel; program name only from no pane or orch pane — held.
16. D2 githubclient credential dir precedent; machine-layer keys per hub — held.
17. D2 bound "any process of the user can read the file … reaches no agent's environment or prompt" — held (bound states the limit itself).
18. D6 notice to a beam orch without a listed strand waits, not refused — held.
19. D8 "dobby already reads each running child's status on its passes" — held.
20. D8 wait start = `At` of "the history entry that made that row current" — **broken** (ambiguous across entry kinds).
21. D8 bound "a resume that re-enters the row restarts the wait's clock" — **broken**.
22. D8 bound "a restarted dobby may remind once early" — held.
23. D8 Beam-Await in `loomshed.OperatorWaitRows` — held.

Findings:

1. **BLOCKING · design · decision-record.md D3 bullet 2, D6 bullet 2, D3 acceptance criterion** — The relay is not outside the starter's pane tree, so its `SLACK` claim fails when a non-orch strand starts it. Counterexample: an agent strand's pane runs `lym slack away`; `proc.Detach` sets only `Setsid` (`internal/proc/proc_linux.go` `Detach`), so the relay's parent stays `lym slack away`, which waits up to 30 s for the first read; creel walks the peer's PPID chain (`internal/creelengine/identity.go` `paneOfPeer`), finds that strand's pane, and `resolveSender` refuses the program claim ("only an orch strand's pane may claim a program name", `identity.go` `resolveSender`). `away` then times out. The same race hits `lym slack start` whenever the relay's first read precedes the starter's exit; the criterion "creel takes its SLACK claim when `lym slack start` runs from a strand's pane" cannot be met as designed. From an orch pane the claim succeeds only because orch panes may claim program names, not because the relay is outside the tree.

2. **MEDIUM · design · decision-record.md D5 bound** — "creel's seven-day retention bounds how long mail waits" is false: retention carries every unread message, and a program's read (keeping its mailbox known), forward (`internal/creelengine/journal.go` `retain`; `internal/creelengine/doc.go` line 68). Mail to `SLACK` waits unbounded while the relay is down, and a relay started weeks later posts all that stale mail to the phone.

3. **MEDIUM · design · decision-record.md D8 bound and bullet 2** — A resume does not restart the wait's clock. A pause/resume writes the status with the existing history via `persistTransient`/`persistMarked` (`internal/shedengine/run.go` lines 882–895); only outcomes and gotos append entries (`run.go` line 377, `reroute.go`, `goto.go` line 137). A run paused at PR-Review overnight and resumed reads as hours into its wait and is reminded at once. Also "the entry that made the row current" is two different entry kinds — a routed entry is the previous row's outcome (`Producer` = previous row), a goto entry carries `Producer` = the target row — and the record does not say which.

4. **MEDIUM · consistency · decision-record.md D3 bullets 3–4** — `lym slack stop` is undone while presence is `slack`: the prime dobby's next pass starts the relay again (D3 bullet 3). No bound is stated; D3 and its acceptance criterion never say `stop` only holds while presence is `terminal`.

5. **LOW · consistency · decision-record.md D4 bound** — "up to the hold's hour" is not a bound: the same owner holding again renews the hold (`internal/creelengine/doc.go` line 251), so a renewing holder delays the operator's mail without limit.

6. **LOW · design · decision-record.md D4 rationale and Constraints** — The 50/min Tier 3 limit is per app per workspace and one app serves every hub on the machine (D1). At the 5-second floor, 4 hubs make 48 calls/min and 5 hubs make 60, so "stays inside … across the machine's hubs" holds only for four hubs or fewer; the record states no hub count.

Tool calls:
1. Grep of `internal/shedengine` for history appends and `persistTransient`/`persistRouted` — context had `HistoryEntry` and `appendOrFold` but not which writes add entries, which the D8 resume bound depends on.
