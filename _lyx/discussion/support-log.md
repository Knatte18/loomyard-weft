# Support log — webster-audit-recovery

## Interview

The session ran autonomously: no operator answered, so every open point below was settled by auto-pick and is recorded in the question ledger.
The board entry already carried four operator decisions (2026-10-05): the `git reset --hard` deny stays and lyx performs every named reset in Go; the gate bounds its wait on a running background shell, with forks waited on as before, and the source of `python3 -` is closed; no Python deny; the fixer stencil forbids the contract files and accept-audit accepts a contract-file finding once rewritten or absent.
One question was left to Discussion: whether the reset verb also covers fabric's manual weft reset.

Turn 1 — exploration.
Read the board entry, the related entries (`agent-permission-setup`, `fabric-reset-to-sha`, `orch-cycle-hardening`), PATTERN.md and the backgrounds for fabric-git, refusal-way-forward, gitrepo-client-boundary and fabric-destruction-chokepoint.
Read shuttle's wait loop (`pollEventsTick`), `claudeengine`'s background detection and event parsing, the Webster Master, fork and fixer stencils, `AcceptPendingAudit`, `checkSuspectPaths`, the run-exit stuck-reason builder, every Webster way forward that names a reset, `refusal-spec.md`, fabric's `ResetHard` primitive and the weft-reset comments, and `~/.claude/settings.json`'s deny list.

Turn 2 — hung shell.
Found that Master legitimately backgrounds `lyx webster recover-batch` for up to `recovery_timeout_min` (60), so a short bound alone would cut it; settled on a short configurable bound plus caller-declared awaited prefixes (D1).
Found the `board-model` transcript is on another machine, so the cause of the `python3 -` call cannot be traced past its command line; the command shows a heredoc attached to the wrong command, leaving `python3 -` on the Bash tool's open stdin.
Settled on closing default stdin for every agent Bash call through the existing PreToolUse hook (D3), plus a Master stencil line (D4).

Turn 3 — contract-file findings.
The fixer stencil says "never touch `_lyx`" but not the two files by name, while it inherits Master's "write outcome and summary last" instruction.
Settled on naming the files in both fork stencils and one evidence function for contract paths used by every checkability site (D4, D5).

Turn 4 — reset verb and scope.
PATTERN-fabric-git and the destruction chokepoint forbid Webster from running `reset --hard` itself; `Fabric.ResetHard` refuses on any tracked dirt and admits the prime, which does not fit the operator's guard.
Settled on a new destroy.go primitive with a caller-supplied own-path exemption and pair-branch ownership, driven by `lyx webster reset --to start|pre-fix` (D6), and on persisting the gate's pre-fix head so `pre-fix` is a recorded commit.
Fabric: the weft reset appears only in Go comments, not in any refusal, stencil or spec, so the verb stays Webster's (D9).

Turn 5 — text and invariant.
Settled the message shape for #362/#365 (D8) and a new scanned PATTERN entry for the operator's rule (D10).

## Rejected alternatives

- **One long shell bound above `recovery_timeout_min`, no declared prefixes.** Keeps shuttle simpler but leaves a hang at over an hour.
- **Bound only shells, never Monitors.** Master never starts a Monitor; treating it as a fork would let a stuck Monitor hang the gate the same way.
- **Kill the expired shell from lyx.** Needs process discovery inside the Claude session (provider-specific) and the pane teardown kills it at run end anyway; the operator asked only for the message.
- **Deny `python`/`python3` in the hook** (as `agent-permission-setup` proposed). Rejected by the operator: Loomyard develops Python in other repos.
- **Hook denies interpreters reading stdin with nothing attached.** Needs shell parsing and covers only interpreters; kept only as the R1 fallback.
- **Stencil-only fix for the hung shell.** Does not close the class for other agents or other commands that read stdin.
- **Run-exit audit downgrades a fork-contract-write already superseded by Master's later write to a policy warning.** Avoids a halt but goes past the operator's decision, which names accept-audit as the clearing step.
- **Webster calls the existing `Fabric.ResetHard`.** It refuses on every tracked dirty path, including the run's own, and its ownership admits the prime, so it cannot refuse on the parent branch.
- **Webster runs `git reset --hard` through `gitrepo` directly.** Violates PATTERN-fabric-git and PATTERN-fabric-destruction-chokepoint.
- **`run --fresh` performs the reset implicitly.** Hides a destructive step inside a run verb and does not serve the `pre-fix` route, which keeps the batches.
- **`lyx webster reset <sha>` with any SHA.** Wider than the operator's guard (a commit the run recorded); every refusal needs only `start` or `pre-fix`.
- **Loosen the agents' deny list for lyx's recoveries.** The operator decided the deny stays.
- **Reset verb covers fabric's weft reset too.** No fabric refusal names it, the guard differs (a remote ref, discarding local weft commits), and `fabric-reset-to-sha` owns fabric resets.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, operator): does the `git reset --hard` deny stay? — Yes; lyx performs resets in Go.
- Q2 (resolved, operator): deny Python? — No.
- Q3 (resolved, operator): how is the hung shell handled? — Bounded configurable wait on shells, forks unchanged, message to the driver, source closed.
- Q4 (resolved, operator via hub recommendation): contract-file findings? — Fixer stencil forbids them; accept-audit accepts once rewritten or absent.
- Q5 (auto-pick): reset verb scope, Webster only or fabric too? — Webster only (D9): fabric names the reset only in comments, and its guard differs.
- Q6 (auto-pick): bound default and shape? — `background_shell_wait_min: 10` in `shuttle.yaml`, plus caller-declared awaited prefixes; Webster declares `lyx webster recover-batch` (D1).
- Q7 (auto-pick): does a Monitor count as a shell? — Yes (A3).
- Q8 (auto-pick): what does the driver receive? — Envelope warning, summary section, friction note, log line (D2).
- Q9 (auto-pick): where is the hung-stdin source closed? — Default stdin `/dev/null` via the PreToolUse hook's updated input, plus a Master stencil line (D3, D4); hook support verified at Plan time (R1).
- Q10 (auto-pick): what clears a contract-file finding, and where does the rule apply? — Absent, or Master's later write; every checkability site (D5).
- Q11 (auto-pick): does the run-exit audit still halt on a fork-contract-write? — Yes (A5).
- Q12 (auto-pick): reset verb shape? — `lyx webster reset --to start|pre-fix`, through a new destroy.go primitive, pre-fix head persisted (D6).
- Q13 (auto-pick): what are "the run's own" uncommitted changes? — Tracked paths the run's transcripts record a write to (D6, R4).
- Q14 (auto-pick): exact sequence for a shed-driven run after `run --fresh`? — Left to the Plan producer to read from the code; the decision fixes only the shape (numbered, each step distinct) (D8).
- Q15 (auto-pick): enforce the operator's rule mechanically? — Yes, new `PATTERN-no-denied-recovery` with a scan test over a closed denied-command list (D10).
- Q16 (open, environment): the `board-model` Master transcript is on another machine (`/home/knatte/...`) and not readable here, so the reason Master reached for a script is unknown beyond its command line.
