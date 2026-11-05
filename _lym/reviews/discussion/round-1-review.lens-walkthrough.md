```yaml
lens: walkthrough
outcome: reported
```

Walkthrough list (decision — case — purpose met?):

- D1 — hubs `ly` and `cs`, each relay reads its own channel — met.
- D2 — `login` reads token on stdin, `auth.test` ok, writes `slack.json`; `setup` checks a private channel and user, writes ids to `<Board>/.lym/config/slack.machine.yaml` (per hub per machine, `configengine/doc.go:42`) — met.
- D3 — operator runs `lym slack away` from a pane — **not met** (F1).
- D4 — case 1: first start, crash, restart with a message in the gap — **not met** (F2); case 2: `@beam1 fix x` reaches `<sh>:beam1:orch` as `[slack] fix x` — met.
- D5 — `<sh>:beam1:orch` replies `--reply` to the `[slack]` message → reply to SLACK, posted with `@beam1` hint — met; beam dobby reminder — **partly missed** (F4).
- D6 — Slack text `back` sets `terminal`, mails `[slack presence]` to every orch, next resume pointer has no line — met (but `away` from a non-orch strand pane fails per F1).
- D7 — `[slack]` mail arrives while presence `slack`; orch replies and mirrors status reports — met.
- D8 — child reaches PR-Review, is paused, resumed 40 minutes later at PR-Review — **not met** (F3).
- D9 — `lym slack` mounted with its verbs — met.

Findings:

F1 — MEDIUM — design — decision-record.md D3 (and D6 `away`). D3's "started detached in a new session … outside every pane's process tree" does not hold while the starter lives: `proc.Detach` only sets `Setsid`, the relay's parent stays the `lym slack away`/`start` process in the pane, and `away` waits up to 30 s for the relay's first read, so that read runs while the starter is alive. Creel walks the PPID chain (`paneOfPeer`) and finds the starter's pane: from a non-orch strand pane the `SLACK` program claim is refused ("only an orch strand's pane may claim a program name") and `away` times out; from an orch pane it passes only by the orch pane's privilege. D3's acceptance criterion ("runs in no pane's process tree, so creel takes its SLACK claim when `lym slack start` runs from a strand's pane") tests a premise the code does not give. Evidence: `internal/proc/proc_linux.go:250-252`, `internal/creelengine/identity.go` `resolveSender`/`paneOfPeer` (lines ~54-66, ~117-145), `internal/dobbyengine/start.go:263-273`.

F2 — MEDIUM — design — decision-record.md D4 cursor. With no cursor a first start "starts from the current time", but the cursor is written only after a message is handled. Walk: relay first starts at T0, handles no message, dies at T1; the operator writes at T2; the prime dobby restarts it at T3 — still no cursor, so it starts from T3 and the T2 message is never forwarded. This is exactly D3's dead-relay-while-away case the restart exists for. Fix: persist the start-time cursor at first start. Evidence: decision-record.md D4 bullets 1-2, D3 bullet 3.

F3 — MEDIUM — design — decision-record.md D8 wait start and bound. History entries are written only when a producer's call ends, named after that producer; the entry that "made the row current" is the previous row's Done entry, or a Goto entry named after the target — two shapes the record does not distinguish. A pause exit or transient halt at PR-Review leaves history unchanged, and a resume that re-enters PR-Review writes no new entry, so the clock still counts from the old entry and a reminder fires immediately after the resume — contradicting D8's bound "a resume that re-enters the row restarts the wait's clock". Evidence: `internal/shedengine/run.go:375-389` (append at call end with `Producer: def.Name`; pause path copies history unchanged), `internal/shedengine/entry.go:25`, `internal/shedengine/goto.go:137`.

F4 — LOW — design — decision-record.md D5 × D8. A beam dobby's reminder comes from the program `<sh>:<slug>:DOBBY`; D5 adds the `(answer: @<slug> …)` hint only to a beam orch's posts, so an operator answering the reminder in Slack reaches the prime orch, not the beam's. Evidence: decision-record.md D5 bullet 2, D8 bullet 3; `contracts/orchrole/orch-role.md:112` (`<shortname>:<slug>:DOBBY` sender form).

Tool calls:
1. Grep for the PR-Review producer files and `HistoryEntry{` writers — context lacked where shed writes history entries.
2. Read `internal/shedengine/run.go:360-395` and `persistTransient` — context lacked the call-end append and the pause path that leaves history unchanged.
