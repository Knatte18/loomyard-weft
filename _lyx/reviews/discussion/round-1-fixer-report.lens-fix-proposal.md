```yaml
lens: fix-proposal
outcome: reported
```

Fix-proposal lens: challenging the reviewer proposals the fixer carried out.

Proposals challenged: F1, F2, F4 (option b), F5, F6, F7, F8 (option a), F11, F12, F13, F14, F15 (F3 was carried out on its judgement-only branch, which leaves nothing to challenge).

1. **MEDIUM** — D5's new halt way forward, "`lyx drift capture` followed by `lyx loom resume`", together with D4's "Run by a caller at a `Drift-Await` halt, `lyx drift capture` runs in the caller's foreground".
   Summary: the F4(b) recapture route fails whenever the capture outlives the caller's process, because `StartTied` kills the command when the capture process dies.
   Evidence: `internal/proc/tied_linux.go` sets `Pdeathsig = syscall.SIGKILL` on the child, and D4 now starts the command through `StartTied`.
   The parent, `ly:orch`, or the arbiter carries out the halt's way forward through a Bash tool call, and that call is bounded (default 120000 ms, at most 3600000 ms).
   A Models-sized capture that runs past that bound is killed together with its command.
   It leaves a `running` record whose lock is free, and the next `Drift-Await` counts that as failed, so the route loops.
   A capture the operator starts this way is also not registered through `runproc.ChildRecord`; D4 registers only the row's detached child.
   So the new D5 adopt paragraph's "the run's processes on the old machine are ended, a running capture included" does not reach it, and neither does the Constraints line about `pairteardown`/`runproc.EndSparing`.

2. **MEDIUM** — D5's changed bullet "capture `running` with its lock held — wait … with no bound of its own: the capture enforces its own timeout (D4) and releases the lock on exit".
   Summary: F6's proposal removes the only halt left for a capture process that never exits after its timeout kill.
   Evidence: `killPlatform` in `tied_linux.go` sends SIGKILL only to `-PGID`.
   A descendant that calls `setsid` or `setpgid` escapes that kill and keeps the command's output pipe open; daemonised build servers, such as `dotnet` build-server nodes, do this.
   In that state `exec.Cmd.Wait` blocks on the open pipe, the capture never reaches "records `failed` once the command is gone", and it holds the lock forever.
   `Drift-Await` then waits with no bound and no halt.
   The old "timeout_min plus one minute" bound was flawed, but it was the backstop.
   Something like a `WaitDelay` on the command, or writing the log to an `*os.File` so no copy goroutine holds `Wait`, is needed to make "the capture enforces its own timeout" hold.

3. **LOW** — D5's new adopt paragraph and the added acceptance criterion "After an adopt on another machine…".
   Summary: F5's proposal covers only loom, so darn's adopt case goes unstated.
   Evidence: darn has no `Drift-Await`.
   After an adopt in darn, the capture record and baseline are equally missing, and D4's main-capture guard (only `Drift-Baseline` or a halted `Drift-Await`) offers no recapture.
   `Drift-Verify` then halts naming `Code-Bouncer`, not `lyx loom goto Describe`.
   The paragraph and the criterion say nothing of darn.

4. **LOW** — the F8(a) text "After `lyx loom goto Webster` past a failed or missing capture, `Drift-Verify` halts a second time, naming `lyx loom goto Describe`", read with D6's new "Under scope `full` every halt over drift … has a reason opening with `shedadapters.DecisionHaltPrefix`".
   Summary: under scope `full`, this no-baseline halt is not "over drift", so it is no decision halt, and the arbiter can settle it with the goto to `Describe`.
   That lands a task meant to change nothing with no compare and no parent ruling, which defeats the parent-only intent F1's fix states in D2 ("no drift in a task meant to change nothing is accepted before the parent rules").

Further tool calls:
- Read `internal/proc/tied_linux.go` and ran `git check-ignore`: my context lacked the Linux kill and parent-death behaviour of `StartTied`, and I wanted to test F15's exclude claim.
- Read the git common dir's `info/exclude` and the `.lyx` link: confirmed F15's claim holds (`/.lyx` is excluded, and `.lyx` is a link into the weft).
