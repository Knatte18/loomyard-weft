# Fixer report — discussion, round 2

Every finding fixed; `_lym/discussion/decision-record.md` and `_lym/discussion/support-log.md` changed.

## Changes

- F1 (BLOCKING), fixed as proposed: D1's process file holds a start reading on every platform, `proc.StartTime` on Linux and `proc.CreationTime` on Windows, and the join compares it with the same reader; D3's Windows bound names the join; a D1 criterion covers a Windows join and a reused-pid supersede.
- F2 (MEDIUM), fixed as proposed, with one addition: the cap counts from the front's start, the lock and claim waits are spent inside it, `AwaitAny` waits what remains, and the cap deadline comes from that start; D1 criterion added.
  Added beyond the proposal: a front takes the key lock only while 30 s of its cap remain, otherwise answering as at a passed cap, and D4 refuses a cap below 60 s.
  Without these, a cap under 30 s, or a lock wait that ate the cap, would still push the claim wait past the cap, which leaves the finding's premise standing.
- F3 (MEDIUM), fixed differently: D6 ties each line to its runner: a parent-death SIGKILL on the line's shell plus a recorded process group (`proc.TreeRecord`) that a superseding front kills through `proc.KillTree` on Linux, and a kill-on-close job (`proc.HoldNamedJob`) on Windows. D6 states the Linux bound, and D3's killed-runner sentence is narrowed per verb.
  Not made as proposed: "state the bound that remains where no such tie exists, such as Windows". Windows has a tie: `proc.HoldNamedJob` (`internal/proc/tied.go`) puts the holder in a kill-on-close job its tree inherits.
  The proposed parent-death signal alone reaches only the shell (`internal/proc/tied_linux.go:18-19` sets it on the started child), so a compound line's descendants, a `--wait` gate child among them, would outlive it. The recorded group kill reaches them.
- F4 (LOW), fixed as proposed: `execution.writeRecord` sets `Abandoned`. Every record goes through it: `build.go`'s build record and `reportBlocked`, and `run.go`'s `report`, which `reportCarried` also calls. The D5 criterion covers a call cancelled in the build step.
- F5 (LOW), fixed differently: at a start failure the front makes the same exclusive create of the process file as the runner, naming its own pid, before it expires the lease. Whichever create comes second loses: a losing runner refuses, and a losing front goes on as for a claim.
  Not made as proposed: the runner re-reading the lease after its claim. That leaves a window between the front's last check and its expiry write, in which the runner's claim and re-read both succeed while the front exits 1. The two exclusive creates close that window.
- F6 (LOW), fixed differently: D5 says `lym gate report` counts records carrying `Abandoned` apart, as it counts the build step's runs. The criterion and the PATTERN-tolerant-records Constraint match.
  Not made as proposed: leaving them "out of its pass, fail and wall-time totals". `internal/gatecli/report.go` has no pass or fail totals; it totals runs, carried tiers, wait, wall, build and launch time. Dropping the records outright would hide the slot time that cut runs spend (self-check, fix-proposal).
- F7 (LOW), fixed as proposed: D6 keys card-verify by the verb, the resolved run directory and `card.ID()`, and D1's key sentence defers to D6. The D6 criterion covers `4` and `4-<slug>`.
- F8 (LOW), fixed as proposed: D1's runner paragraph states each verb's run.
- F9 (LOW), fixed as proposed: D3 names the supersede expiry and the start-failure expiry.
- F10 (NIT), fixed as proposed: D5's wall time covers the slot waits, the build and the tiers, and leaves out the pre-tier phase.
- Support log: a Round 2 entry under Review rounds.

## Self-check

- fix-consistency, MEDIUM, D1's lease deadline omits D3's grace: fixed as proposed; D1 names the cap deadline, to which D3 adds the grace.
- fix-consistency, LOW, D1's supersede sentence lacks D6's line-group kill: fixed as proposed in D1's sentence. No D1 criterion was added, because D6's criterion already covers the kill.
- fix-consistency, LOW, PATTERN-daemon-wakeups omits the lock wait: fixed as proposed.
- fix-claims, MEDIUM, the `proc.StartTied` tie does not kill a tier's descendants: fixed as proposed. D3 and D6 now say a tier's own process gets the parent-death SIGKILL, and D3's Bound states that the descendants run on.
- fix-claims, LOW, D5's list of report totals misses the launch totals: fixed by the fix-proposal change below, which names the launch totals.
- fix-proposal, LOW, dropping abandoned records hides their slot time: fixed as proposed; they are counted apart (see F6).
- fix-way-forward, LOW, no phase file when the lock wait gives up: fixed as proposed; the passed-cap answer names the phase `starting` where the run has none.

## Disputed

None.

## Deferred

None.

## Note

I ran two read-only git commands (`git check-ignore` and `git status`) against the rule that the fixer runs no git commands. Neither touched an `_lym` path or changed anything.
