```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked each claim, bound and guarantee in the decision record. Each line below says whether it held or broke.

**Findings**

1. **BLOCKING (D13 `deploy`, D11): any agent can get host code execution through `deploy`.**
   - Evidence: D11 hands the area a token with push reach to the hub's repositories, and D5 puts the supervisor's socket on every process's allow list.
   - Counterexample: an agent commits a hostile `tools/deploy/main.go`, pushes it to the parent branch on origin with the area token, then connects to the supervisor and calls `deploy <that sha>`. That sha is origin's tip, so the supervisor builds and runs it on the host.
   - Nothing requires the token to lack push to the parent branch, and nothing requires the commit to have come through a landing. "deploy runs only origin's tip" therefore bounds nothing.
   - D16's non-claim covers landed code, but this commit was never landed.
   - Possible fixes: accept `deploy` only from the Land row's own process, require branch protection on the token, or require operator approval.

2. **MEDIUM (D13 `hibernate` and `window`): the verbs do not limit who calls them or when.**
   - Evidence: the supervisor socket is open to every agent. `lym hibernate request` pauses every session before it calls the verb, but the verb itself checks no pause.
   - Counterexample: an agent calls `hibernate` directly and hibernates the host in the middle of other runs. It can also call `window` in a loop and open Konsole windows on the host.
   - Both are actions outside the area, which the threat model says the area prevents. D13 bounds what each verb runs but not who may call it.

3. **MEDIUM (D9 `RunMemoryCapped`): adopting the pid after start leaves its early children uncapped.**
   - Evidence: D9 says "starts its command and has the supervisor adopt the pid before it waits". Moving a pid into a scope does not move children it has already forked.
   - Counterexample: `go test` forks the compiler and test binaries within milliseconds. Children forked before the adoption stay in the pane's cgroup without this cap.
   - Today, `systemd-run --scope` places the command in the scope before it execs.
   - Possible fix: cap a held child, or a shim that waits on a pipe, before it execs the command.

4. **MEDIUM (D12 against D7): the relay verb is refused by D7's own rule.**
   - Evidence: the relay is lym code started outside the area for the hub. D7 refuses every verb outside the area whose cwd resolves to an area hub, except `lym area`, `reed attach`, `version` and help.
   - The relay's verb is not on that list, so D7 refuses it. Otherwise D7 needs a further exemption, and that exemption is a host-side lym reading hub files.

5. **MEDIUM (D12): creel's own docs contradict "existing creel verb boundary already confines what it delivers".**
   - Evidence: `internal/creelengine/doc.go` says "any of them can make the `SLACK` claim and take, settle or deliver bridge mail".
   - If the "them" of that line includes processes in the hub's panes, an agent in the area can claim `<shortname>:SLACK` and deliver mail stamped from another machine's orch.
   - D12 rests on a boundary the code says is weak. I did not trace which processes "them" covers.

6. **MEDIUM (D8): daemons still running an older binary make the third bullet's guarantee false.**
   - Evidence: the guarantee needs every receiving daemon to recognise a peer inside another area. D5 allows connects to other hubs' creel sockets from any process in the area.
   - Counterexample: hub B's daemon, started before the binary change, reads `SO_PEERCRED`. That names A's supervisor, a host process in no pane, so B treats the agent's post as `EXTERNAL`, which may inject into and interrupt any strand of B, its orch strand included.
   - The record does not require other hubs' daemons to restart, or a minimum daemon version, before the derived list admits their sockets.

7. **MEDIUM (D2, D3, D10, D13): only the area file is protected from writes from inside.**
   - Evidence: D2's start check refuses only a write grant "on the file or its directory".
   - Counterexample one: `write: [~]` or `[~/.config]` grants an ancestor. A literal check of the file or its directory passes it.
   - Counterexample two: `write: [~/go]` lets agents rewrite the `lym` binary that the next supervisor and `deploy` run on the host.
   - Other host-side inputs are not placed outside D10's writable grant either: the supervisor's pid record (D3), the deploy clone (D13), the token file and the hub record. All live under `~/.local/state/lym` or `~/.config/lym`.
   - D10 grants every right on the area's engine config directories under the same state root `~/.local/state/lym`.
   - A deploy clone that lands under a granted path can be poisoned. An agent sets `core.fsmonitor` or a hook, and the supervisor's `git fetch` then runs agent code on the host.
   - The start check needs to cover every host-side input and every ancestor grant.

8. **MEDIUM (D7): one root hook does not stop other root-level work that runs first.**
   - Evidence: `cmd/lym/main.go`'s root `PersistentPreRunE` already runs work such as the stencil seed pass (`cmd/lym/stencilseed.go`) for every command. That includes the exempt `lym area` and `reed attach` verbs.
   - D7's claim that host-side area verbs "read no hub config" needs every such root step to skip or come after the area check for exempt verbs. The record does not order the new check against them.

9. **LOW (D3, `lym area stop`): stray orphans survive stop and can still write the hub.**
   - Evidence: `stop` ends processes through `runproc` and then ends the supervisor. An agent's orphan with no runproc record survives.
   - After stop, such an orphan keeps its Landlock grants on the hub and can still write it. Only its connects fail with `ENOSYS`.
   - So "stop" does not mean no agent code still writes the hub.

10. **LOW (D15): the profile-path check may not be possible without privilege.**
    - Evidence: the start "refuses when the profile ... names other paths than the file and the hub's geometry give".
    - Loaded AppArmor policy is compiled, and an unprivileged process may not be able to read it back as paths. A hash in the profile name would be checkable.
    - Not checked.

11. **LOW (D9): the pidfd order is not stated, so pid reuse can beat the check.**
    - Evidence: a check of ancestry through `/proc` followed by `pidfd_open` can race with pid reuse. The pidfd must be opened before the ancestry check.

**Claims that held**

- D4 fails closed when an agent makes itself non-dumpable: `/proc/<tid>/mem` and `pidfd_getfd` then fail and the connect is refused.
- An agent cannot signal or ptrace the supervisor.
- Foreign ABIs, `io_uring` and abstract addresses are refused.
- D4's datagram bound is stated.
- D6 fails closed: an agent that unsets `TMPDIR` meets Landlock's denial on `/tmp/lym-<uid>`. The socket paths fit under 108 bytes.
- D6 and D3 inside-detection: a host process with its own seccomp filter is told apart by the supervisor-pid ancestry.
- D8: forged `SCM_CREDENTIALS` is refused by the kernel. With `SO_PASSCRED` set, credentials are always attached.
- D8: a forged `deliver` is caught by `confirm` against the sending daemon.
- D3: other hubs' outbound workers starting this hub's daemon are refused by the spawn site.
- D9: an agent gains no bus call. Capping its own process tree or the creel daemon is harmless.
- D14 order: creel identity lands before the boot.

**Extra tool calls**

- Grepped the root `PersistentPreRunE` and `hubreconcile.Ensure` callers, to test D7's root-hook claim; my context did not show what root pre-run work exists.
- Grepped how `creelengine` identifies the `SLACK` program, to test D12's boundary claim; my context had no creel identification code.
- Read `standalonestate`'s state-root derivation and the hub-record path, to test whether D10's writable state directory overlaps D3's, D8's and D13's host-side inputs; my context had only the function names.
