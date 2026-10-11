# Discussion round 3 — fixer report

All three findings were validated and fixed; `lyx loom validate-discussion` exits 0 on the result.

## Fixed

- **F1 (BLOCKING, design)** — confirmed on tmux 3.6: the guard invocation with a file holding `set -g bogus-option x` exits 1 with `invalid option: bogus-option` while `exit-unattached` and global `destroy-unattached` both read `off` and the line after the error applied.
  - D4's trigger is now "a guard invocation that ran `source-file` exits non-zero", and D4 states that this exit status never decides a pin failure.
  - D4's Bound names the guard invocation's status as the non-fatal one and points at D5 step 3.
  - D5 step 2 drops "fatal like `remain-on-exit`" and says its exit status is D4's warning.
  - D5 step 3 now also reads back the global `destroy-unattached` and the server's `exit-unattached`, failing the boot like a failed `remain-on-exit` pin when either is not `off`.
  - D5's rationale records the probe result that motivates the read-back.
  - New acceptance criterion: a file with a run-time error that also sets `exit-unattached on` and `destroy-unattached on` boots, warns, and leaves both options `off`.
  - Support log: a rejected alternative "Fail the boot on the guard invocation's non-zero exit", and Q8's resolution names the read-back.
- **F2 (LOW, design)** — the premise is restated in D3's concurrency paragraph and in D5's rationale: tmux runs a client's queue without interleaving others until a command waits.
  D5 names `run-shell` without `-b` (tpm's last line) as the waiting case, and says step 1's chaining, the non-waiting claim and the still-connected guard client keep that window harmless.
  D3 adds that the claim's check-and-set waits on nothing.
- **F3 (LOW, design)** — D3 step 2's "always goes to the server's first starter" now reads "whenever that starter reaches its guard invocation".
  D3's Bound gains the residual case: a server whose starter's boot ends between spawn and guard invocation carries no claim, has no later candidate, and runs without the file until its next start, which the last worktree's `down` brings about.

## Disputed

None.

## Deferred

None.
