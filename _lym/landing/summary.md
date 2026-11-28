# Study running gate tier runs on a WSL2 machine on the LAN

Adds `docs/research/gate/septim-farm-study.md` and its entry in `docs/research/README.md`.
No code changes.

The study asks how much work, and how realistic, it is to run `lym gate test` tier runs on Septim,
a Windows 11 desktop with WSL2, while the hub stays on Neve.
It answers for a stored mode (code checked out there) and an ephemeral mode (code held in RAM for one run).

## Findings

- **Stored mode is realistic:** four code tasks and one setup and trial task.
  Only the tier's commands should run remotely, cut at `runInvocations`,
  so scope, green record, counter, records and reports stay with the caller.
  Cleanup, the watchdog and memory guards, the `lym` prebuild and the stray-file check must run on the remote.
  The remote needs its own slot pool.
- **Ephemeral mode is only partly reachable:** WSL2 settings can keep a run from writing code to disk,
  but cannot stop Windows from paging or hibernating the VM's memory.
  It adds two tasks, three with a C# hub.
- **Speed is unmeasured.** Nothing was run on Septim, so both recommendations hang on a measurement.

## Recommendation

- Stored: measure `tier1` and `tier2` on Septim by hand first, then build it, keeping `tier0` local.
- Ephemeral: do not build it yet; settle the disk guarantee, whether work code may leave its laptop,
  and whether a kept RAM cache counts as stored.

## Limits

The study reads the code at `4a19e81cf`, existing gate run records, earlier reports and 12 public pages.
Claims no page states are marked unverified in the report, among them Wake-on-LAN, host sleep, Defender
and starting WSL at boot.

## Merge danger

**Door:** two-way
**Blast radius:** docs
