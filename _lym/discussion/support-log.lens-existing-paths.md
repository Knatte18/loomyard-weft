```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens review of the reed-slack-relay decision record and support log.

**Paths traced, by decision**

- **D3, relay process ownership.**
  - `lym orch stop` and the beam-down path end only `RoleDobby` under `HubOwner` (`internal/orchcli/cli.go`, `endDobby`). The relay survives them, but no path ever ends it.
  - The tender sweep never treats `HubOwner` records as orphans (`internal/tenderengine/processes.go`, `orphanOwner`). The relay survives the sweep, and no path ends it there either.
  - When Land deploys a new binary, the relay survives through its re-exec.
- **D3/D4, the creel sender check.** The creel daemon walks the relay's process ancestry to decide who is posting.
  - Once the starting `lym` exits, the relay is outside every pane, so its program claim is accepted. That holds.
  - While the starting process is still alive inside a non-orch strand's pane, the relay counts as that strand, and creel refuses its program claim. This is a narrow window, and the creel daemon's own detached start has the same shape.
- **D4/D6, mail to a paused or held orch.** Creel stores the post, but a `lym tender pause`, a hibernate pause or another sender's hold holds back delivery into the orch. Breaks; see the MEDIUM finding.
- **D5, the program mailbox.**
  - A reply to a `[slack]` message reaches the program mailbox by the reply-to-sender rule, provided the relay has read once. Survives.
  - Program mail is never chased or reminded, and retention carries it forward. Survives.
- **D6, presence notices to every beam orch.**
  - When a beam's orch strand is not listed, program mail to it waits in the pool and arrives later, in order. Survives.
  - The resume pointer covers both the hook path after compaction and the pointer typed at a refresh. Survives.
- **D8, the dobby reminder.**
  - Dobby reads the child's current row through the same `WaitsOnOperator` path batten uses. Survives.
  - The wait's start comes from "its status record". I did not verify that the status history carries a timestamp for when the row was entered.
- **D2, machine-layer config.** `lym config reconcile` prunes only base-layer keys and never touches the machine layer. Survives.

**Findings**

- **MEDIUM — D4/D6: a hub pause or hold silently delays Slack mail.**
  - While `lym tender pause` or a `lym hibernate request` pause covers the orch, creel holds every delivery into it, including through its hooks. A hold from another sender does the same for up to an hour.
  - The relay gets the post's id and moves its cursor on, so the operator on the phone gets no answer and no sign that the hub is paused.
  - The record says nothing about this, and `lym slack status` and the Slack `status` command do not report a pause.
  - Evidence: `internal/creelengine/doc.go`, sections Pause ("post still stores the mail") and Holds.
- **MEDIUM — D3: nothing stops the relay, and it has no role.**
  - The record gives the relay no `runproc` role and no stop verb. The existing stop paths end `RoleDobby` only, and the tender sweep never treats a `HubOwner` record as an orphan.
  - So once started, the relay can only be killed by hand, which `PATTERN-run-process-owner` forbids ("ended only via runproc's End, EndSparing or EndRole").
  - Evidence: `internal/orchcli/cli.go`, `endDobby`; `internal/tenderengine/processes.go`, `orphanOwner`.
- **LOW — D3/D4: the relay's sender identity depends on who started it.**
  - Creel decides the sender by walking the relay's process ancestry. If an agent's pane runs `lym slack away` and the starter is still alive when the relay first posts, creel sees that strand and refuses the program claim.
  - The bound is unstated.
  - Evidence: `internal/creelengine/doc.go`, Sender identity.
- **LOW — D4: mail between 4 KiB and 8 KiB is cut to a reminder on a pane-mail orch.**
  - Creel types a notice whose body is over 4 KiB into a pane-mail orch, such as a Codex orch, only as a reminder line.
  - D4's cap is creel's 8 KiB line limit, so a long Slack message can arrive at such an orch as a bare reminder.
  - Evidence: `internal/creelengine/doc.go`, Mail ("A notice … whose body is at most 4 KiB").
- **LOW — D5/D6/D8: posts to the `SLACK` mailbox are refused until the relay first reads.**
  - Creel accepts a program as a receiver only after it has read once. If the relay fails before its first read, for example on a bad token after `lym slack away`, dobby reminders and orch posts to `<shortname>:SLACK` are refused.
  - The record does not say whether those posts are refused or queued in that case.
  - Evidence: `internal/creelengine/doc.go`, Receivers.
- **LOW — D2/D3: two machines configured with the same channel both relay it.**
  - Each machine's relay polls the channel with its own cursor, so every operator message reaches both machines' orchs.
  - This is possible for a hub that runs on both Neve and the work machine, since the keys are machine-layer and nothing refuses a channel already in use.
  - The record states no bound.

**Tool calls**

- Grep for `HubOwner` users and `agentname` program-name validation: my context had no list of what ends `HubOwner` processes, or of the program-name rule for `SLACK`.
- Read `tenderengine/processes.go` `orphanOwner`, the `agentname` program-name validator and the `orchcli/cli.go` `endDobby` owner: to confirm which roles the existing stop and sweep paths end.
